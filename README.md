# PodFit - Kubernetes Resource Calculator

A small DevOps capacity-planning tool built with plain HTML, CSS, and JavaScript. The complete application lives in [index.html](./index.html), with no frameworks, backend, database, or external dependencies.

## Features

- Multiple workloads with CPU requests in millicores, memory requests in MiB, and replica counts.
- Configurable node resources, maximum pods per node, and resource buffer (default **20%**).
- Live node estimates, per-workload pods-per-node capacity, and CPU/memory limiting-resource badges.
- Cluster utilization bars and a mixed-workload pod placement preview.
- Dark dashboard with responsive desktop and mobile layouts.
- Inline validation for missing values, negative resources, invalid buffers, and replicas below 1.
- Warnings when a complete pod cannot fit on a node.
- Sample values and a Reset button that clears inputs and restores the default buffer.

## Run locally

1. Download or copy `index.html`.
2. Open it directly in a modern web browser.
3. Edit the preloaded sample values, or select **Reset** to start a new plan.

No installation, build step, internet connection, or local server is required. Changes are not saved when the page is reloaded.

## Calculation logic

Use node resources available to these workloads, after system reservations and existing workloads. Scheduling estimates use pod **requests**, not limits.

```text
usable CPU    = node CPU x (1 - buffer / 100)
usable memory = node memory x (1 - buffer / 100)

pods per node = min(
  floor(usable CPU / pod CPU),
  floor(usable memory / pod memory),
  available pod slots
)

nodes for one workload = ceil(replicas / pods per node)
```

Zero CPU or memory requests do not constrain that resource; the pod-slot limit still applies. The buffer must be a whole number from 0 to 99%.

For mixed workloads, a best-fit decreasing heuristic places complete pods while checking CPU, memory, and pod slots on every node. The node estimate comes from this placement, **not simply dividing total resource requests by node capacity**. Per-workload capacity cards are independent scenarios and must not be added together.

**Example:** For pods requesting 500m CPU and 512 MiB memory, with 10 replicas, on nodes providing 4000m CPU and 8192 MiB memory:

- After a 20% buffer: **3200m CPU and 6553.6 MiB** remain per node.
- CPU allows `floor(3200 / 500) = 6` pods; memory allows `floor(6553.6 / 512) = 12`.
- CPU is limiting: **6 pods per node**, requiring **2 nodes**, with a 6/4 placement.
- A pod requesting **5000m CPU** cannot fit. The app shows an inline warning and hides estimates; adding more identical nodes cannot solve an oversized individual pod.

## Limitations

- A planning estimate, not a guarantee of Kubernetes scheduler behavior.
- One identical node type; no multiple node pools or live cluster integration.
- Mixed-workload packing is heuristic and may not find the minimum possible node count.
- Does not model affinity, taints, topology, storage, rollout surge, or node failures.
- System and DaemonSet overhead must be accounted for in the resource and pod-slot inputs. A 20% buffer does not guarantee node-failure tolerance.
- Whole-number inputs only; maximum 30 workloads and 5,000 total replicas. Placement previews show the first 60 nodes; totals include all nodes.
- No persistence, cloud pricing, or automatic deployment.

## How we used vibe coding

1. **Idea and discussion:** Started with a DevOps need, discussed features and UI, and agreed on multiple workloads, one node type, and per-node resource headroom.
2. **Code generation:** Used an AI assistant to create a single-file browser application from those requirements.
3. **Iterative improvements:** Refined it into a dark dashboard, added pods-per-node insights, clarified whole-pod placement, improved warnings, and added inline validation and Reset.
4. **Testing and verification:** Ran 18 calculation tests plus automated browser checks for editing, validation, reset, and responsive layouts from 320 to 1440 pixels. Verified the exact 500m/512 MiB example and the oversized 5000m case in both the engine and browser.

The workflow combined natural-language requirements, generated code, user feedback, and executable checks rather than accepting the first generated version. Verification scripts were kept outside the application folder to preserve the single-file app.
