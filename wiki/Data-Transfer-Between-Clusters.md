# Data Transfer Between Clusters

How to move large files (feature caches, model checkpoints, datasets) between
our three GPU systems. Added 2026-08-02; conclusions from an external
consultation, recorded without the discussion.

The three systems and their constraints:

| System | SSH access | Best transfer support |
|---|---|---|
| **Anvil** (Purdue RCAC) | Direct SSH, key auth | Globus, rsync/scp/sftp, S3 object storage |
| **Delta** (NCSA) | Password + Duo only, **no SSH keys** | Globus (recommended); scp/rsync for small files only |
| **OrangeGrid** (Syracuse, HTCondor) | Via jump host/bastion | SSH/scp through login node; no published Globus collection |

## The design

```text
Delta  <========== Globus ==========>  Anvil
                                           ^
                                           |
                                      rsync over SSH
                                      (initiated from OG)
                                           |
                                           v
                                      OrangeGrid
```

- **Anvil ↔ Delta:** Globus directly.
- **OrangeGrid ↔ Anvil:** rsync over SSH, initiated from OrangeGrid
  (Anvil is publicly reachable with key auth; OG needs its bastion inbound).
- **OrangeGrid ↔ Delta:** two stages through Anvil (avoids Delta's
  password+Duo SSH, which blocks unattended server-to-server rsync).
- **croc:** emergency/ad-hoc fallback only (see below).
- Large data never routes through a laptop.

## Anvil ↔ Delta: Globus

Globus File Manager with the ACCESS identity; collections **`Anvil ACCESS`**
and **`ACCESS Delta`** (an `NCSA Delta` collection also exists under the NCSA
identity; ACCESS is simpler for us). Delta's collection exposes home plus
`/work/hdd` and `/work/nvme`.

For repeated/resumed transfers enable **"Sync only files where checksum
differs."**

CLI form (asynchronous — the terminal does not need to stay connected):

```bash
globus transfer \
  "$ANVIL_COLLECTION_ID:/anvil/scratch/.../run-2026-07-29/" \
  "$DELTA_COLLECTION_ID:/work/hdd/.../run-2026-07-29/" \
  --recursive \
  --sync-level checksum \
  --verify-checksum \
  --label "features 2026-07-29"
```

## OrangeGrid ↔ Anvil: rsync

One-time setup on OrangeGrid — dedicated transfer key:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_anvil_transfer -C "orangegrid-to-anvil-transfer"
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519_anvil_transfer
```

Add the public key to `~/.ssh/authorized_keys` on Anvil (optionally prefixed
with `restrict`). Then in OrangeGrid's `~/.ssh/config`:

```sshconfig
Host anvil-transfer
    HostName anvil.rcac.purdue.edu
    User x-ACCESSNAME
    IdentityFile ~/.ssh/id_ed25519_anvil_transfer
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 10
```

Push OG → Anvil (run on OrangeGrid; pull is the same with source/dest swapped):

```bash
rsync -aH --partial --partial-dir=.rsync-partial --info=progress2 \
  /orangegrid/path/features/ \
  anvil-transfer:/anvil/scratch/x-ACCESSNAME/staging/features/
```

Rules that matter:

- Trailing `/` = copy directory *contents*.
- `--partial` + `--partial-dir` allow resume without half-written files
  masquerading as complete outputs.
- **No `-z` by default** — compression wastes CPU on checkpoints and
  already-compressed data.
- **Never transfer a checkpoint that is still being written.** Write to a
  temp name, atomically rename on completion, then transfer.
- If outbound port 22 from OG is blocked: bridge via a lab server
  (OG → lab server via rsync/ProxyJump; lab server → Anvil/Delta via Globus
  Connect Personal).

## OrangeGrid ↔ Delta: stage through Anvil

```text
OG → Delta:  (1) OG → Anvil rsync   (2) Anvil → Delta Globus   (3) verify   (4) clean staging
Delta → OG:  (1) Delta → Anvil Globus   (2) OG pulls from Anvil rsync   (3) verify   (4) clean staging
```

## HTCondor: do NOT use job file transfer for datasets

Copy data into OrangeGrid's shared filesystem once; jobs read it there. Omit
`should_transfer_files = Yes` / `when_to_transfer_output = ON_EXIT` — the OG
docs say Condor file transfer is unnecessary on the shared filesystem, and
per-job copies of large feature sets multiply I/O, space, and startup time.

## Many small files: shard first

Metadata operations dominate when transferring millions of small `.npy`/
`.json` files. Package into immutable shards of roughly **10–50 GB**:

```bash
tar -cf - features/part-001/ | zstd -T8 -3 -o features-part-001.tar.zst
```

- Run compression as a **compute job**, never on a shared login node.
- For checkpoints, test whether zstd is worth it first
  (`time zstd -3 model.safetensors -o model.safetensors.zst`) — often it is not.
- Manifest before transfer, verify after:

```bash
sha256sum *.tar.zst *.safetensors > SHA256SUMS   # source side
sha256sum -c SHA256SUMS                          # destination side
```

- Prefer several shards over one enormous archive: better retry granularity,
  independent extraction.

## Staging layout on Anvil

- `$PROJECT` — durable datasets, checkpoints, manifests (5 TB, snapshotted,
  not purged while the allocation is active).
- `$SCRATCH` — transfer staging and active runs only (100 TB, **no backup,
  30-day purge of inactive files**).
- `$HOME` — never for large data (25 GB).

```text
$PROJECT/<project>/{datasets,checkpoints,manifests,README.md}
$SCRATCH/<project>/{active-runs,extracted-features,transfer-staging}
```

Keep at least one durable copy of every expensive-to-reproduce checkpoint
outside scratch. Anvil's S3-compatible object storage (via `rclone`) is the
candidate for a future canonical store, with clusters holding working copies.

## croc: fallback only

Good tool (PAKE-encrypted, resumable, relay-based, no port forwarding), wrong
default for HPC. Use it only when **all** hold: Globus and SSH transfer are
unavailable; admins permit it; the transfer is occasional, not a workflow; no
restricted data through an unapproved relay; run on a data-transfer or
allocated node, not a login node. Pass the secret via `CROC_SECRET`, never on
the command line.

## Bottom line

```text
Anvil ↔ Delta:       Globus
OrangeGrid ↔ Anvil:  rsync initiated from OrangeGrid
OrangeGrid ↔ Delta:  stage through Anvil
croc:                emergency/ad-hoc fallback only
```

One durable canonical location; checksum manifests; transfer only immutable
completed outputs; treat every cluster scratch as replaceable working space,
never as backup.

## References

- Anvil: docs.rcac.purdue.edu/userguides/anvil/file_management/ ·
  getting-started/ · objectstorage/concepts/
- Delta: docs.ncsa.illinois.edu/systems/delta — data_mgmt (transfer) ·
  login (Duo/no-SSH-keys)
- OrangeGrid HTCondor: su-jsm.atlassian.net/wiki/spaces/RESCOMP/pages/164169368
- Globus CLI: docs.globus.org/cli/reference/transfer/
- croc: github.com/schollz/croc

## Related

[[Data-and-Caches]] · [[Anvil-Interactive-GPU-Workflow]] ·
[[Delta-Setup-and-Parallel-Workflow]] · [[Anvil-vs-Delta]]
