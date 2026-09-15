# Securing & Governing the Medallion Lakehouse
*DP-700 · Unit: Secure and govern the lakehouse (need-driven reference)*

## The need
Unit 2 said each layer has an audience. That was a diagram.
Nothing yet *stops* an analyst from querying bronze (duplicates →
inflated revenue in a meeting) or an intern from dropping a silver
table. Security turns "audience per layer" from convention into
enforcement.

## Two levels of access control

### Level 1 — Workspace & item permissions (coarse)
- **Workspace roles** — Admin / Member / Contributor / Viewer —
  apply to EVERYTHING in the workspace. Blunt instrument.
- **Item permissions** — share one specific lakehouse with a
  colleague without opening the whole workspace.
- **Separate workspaces per layer** = strongest wall: own capacity,
  own role assignments, clear ownership boundary. Cost: more
  workspaces to manage. (The walls ladder, final form.)

### Level 2 — OneLake data access roles (granular)
The need: the team shares ONE workspace/lakehouse, but gold
consumers shouldn't even SEE bronze or silver.
- Scope a role to specific **tables or folders** inside a lakehouse.
- Gold consumer queries gold; bronze/silver stay invisible.
- No extra workspaces needed.
- Configure: open lakehouse → **Manage OneLake security** →
  create role → define scope → assign members.

### ⚠️ The DefaultReader trap (exam bait)
Every lakehouse ships with a built-in **DefaultReader** role that
grants ALL ReadAll users access to ALL data. To actually restrict
access you must **modify or delete DefaultReader** — otherwise your
carefully scoped roles sit next to an open front door.

### Which approach when
| Situation | Choose |
|---|---|
| Teams share a workspace, need different table access per layer | OneLake data access roles |
| Strong isolation, compliance boundary, separate capacity | Separate workspaces per layer |

## Manage change with Git

### The need
A medallion pipeline IS code: notebooks, pipeline definitions,
schema definitions — all must stay in sync. Without version
control, one bad deploy corrupts silver mid-layer with no undo.

### Git integration
- Workspace ↔ Git repo: notebooks, pipelines, and lakehouse
  definitions versioned **together**.
- Transformation breaks silver? **Revert to the previous commit.**
- Branches + pull requests — the same workflow as application code.

### Deployment pipelines
- The need: "worked in dev" ≠ safe in prod.
- Promote the workspace **dev → test → prod** in a controlled
  sequence.
- **Compare environments** before promoting — catch differences
  before they reach production data.

## Exam triggers
| Question says… | Think… |
|---|---|
| same workspace, per-layer table access | OneLake data access roles |
| compliance / regulatory isolation | separate workspaces |
| roles created but everyone still reads everything | DefaultReader still active |
| revert a broken transformation | Git integration |
| promote dev → test → prod, diff environments first | deployment pipelines |
| share one lakehouse without sharing the workspace | item permissions |

## Memory hooks
- Permissions ladder: workspace roles (hammer) → item permissions →
  OneLake data access roles (scalpel)
- **DefaultReader = the open front door** — close it before
  decorating the rooms
- **Git = the undo button your pipeline never had**
- Deployment pipelines = **dress rehearsal before opening night**