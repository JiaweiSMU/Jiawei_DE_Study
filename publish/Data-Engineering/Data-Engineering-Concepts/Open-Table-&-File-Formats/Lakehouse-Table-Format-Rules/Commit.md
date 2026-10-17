The moment a ==change becomes part of the table==. Before it, the change exists only as files nobody reads. 
Done via **switching the catalog’s pointer to a new metadata file**.
### Why a single switch is needed
It's to prevent readers from getting half written files.
- In a commit, everything is written first, where no reader can see it and then ==make it visible in one step that either fully happens or doesn't happen at all==.
### The steps
Table `orders` is at `v5`. A job appends today’s orders.
```text
Catalog:  orders → v5.metadata.json
```
Step 1: note the current version.  (`v5`)
Step 2: write the new data files.
```text
data/file-d.parquet   ← new
data/file-e.parquet   ← new
```
- They’re in storage, but `v5` doesn’t list them, so **no reader opens them**. 
Step 3: write the new metadata file.
```text
metadata/v6.metadata.json   ← v5's file list + file-d + file-e
```
- Still invisible as the catalog is still pointing to `v5`, so readers never look at `v6`.
Step 4:  Ask the catalog to switch the pointer, on one condition:
- ==Change `orders` from `v5` to `v6`, **but only if it still points to `v5`**.==
```text
Catalog:  orders → v6.metadata.json      ← committed
```
- If it succeeds, every new query reads `v6` and sees `file-d` and `file-e`. 
### What readers see
No query ever sees `file-d` without `file-e`. That’s **atomicity**: all of the write, or none of it.
### What can go wrong
**1. The job crashes before step 4.**  
The files written in steps 2–3 are left behind as **orphan files**, in storage but in no version, until a cleanup job deletes them.
**2. Another writer committed first.**  
A’s switch is **refused**, so A can never silently overwrite B’s commit.
A then checks what B changed:
- **No overlap** (e.g. both only added files): A builds a new metadata file `v7` on top of `v6`, keeping B’s files and adding its own, then tries `v6 → v7`. Only metadata is rewritten
- **Real overlap** (e.g. both rewrote the same file): the two changes can’t both be applied, so A’s commit fails and the job must rerun.
