# sweep-tasks SLURM templates

Reference sbatch scripts for ibex. Pair each one with the matching YAML
in `examples/tasks/`.

## `build_crg_plan_obn3d.sbatch`

Generate the production OBN CRG plan cache (the `crg_fwi_plan_v1` npz consumed
by `geometry.kind=from_crg_plan` / `obs.kind=crg_plan` in FWI YAMLs).

### Submission

```bash
ssh glogin.ibex.kaust.edu.sa
cd ${SWEEP_RUNS_ROOT}/crg_plan_build
sbatch ${SWEEP_STACK_ROOT}/sweep-tasks/examples/sbatch/build_crg_plan_obn3d.sbatch
```

### Single-process (smaller dataset, local box)

```bash
sweep-tasks build-crg-plan \
    --segy-root <segy-dir> \
    --glob '*.sgy' \
    --out <output>.npz \
    --num-workers 16 \
    --source-depth-m <SDEPTH_const> \
    --receiver-z-byte <Z_BYTE_OFFSET> \
    --receiver-quantize-m 0.5
```

### Dataset-specific overrides

| Dataset | source-depth-m | source-z-byte | receiver-z-byte | receiver-quantize-m | notes |
|---|---|---|---|---|---|
| OBN-3D  | 9.0 (airgun) | not needed (override wins) | 40 (byte 41-44 = gz) | 0.5 | OBN nodes on 375 m grid |
| Viking  | 6.0 (streamer) | n/a | n/a (default ok) | 0.1 | 2-D streamer, no CRG |
| Volve   | -            | n/a | n/a | -      | 2-D OBC; quantize TBD |

The legacy pipeline (`fwi_workflow-dev`) used **MPI 50-rank** at ~24 min
wall on ibex `batch` partition. sweep-tasks matches that timing because
it reuses the same per-file scan logic + an in-memory MPI gather
(simpler than the legacy "sharded-merge" path; works because the production OBN data's
merged per-trace arrays fit in rank-0 RAM well under the partition
default).

### Output schema (matches legacy)

The output npz has format `crg_fwi_plan_v1` and is interchangeable with
the legacy build at
the reference CRG plan npz on disk.
Validation on 10-file subset showed 73/74 receivers matching position
within 0.1 m and identical z-range (159-688 m).
