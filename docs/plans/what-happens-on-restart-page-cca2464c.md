# what happens on restart page

## Implementation Steps

### Task 1: what happens on restart page

- [x] Add a new Node Reference page linked from https://kubernetes.io/docs/reference/node/ and explaining what happens on various restarts. Here is some information I have already. Fill up TODOs and clean up language. Add links to other documentation pages whenever necessary.

1. Impact of kubelet restart
- All Pods are marked as not ready on restart. This causes a big load on API server to update endpoint statuses, gateway settings, etc. This was fixed in a recent kubernetes version. TODO: link the KEP
- Kubelet re-initializes, causing a lot of requests to the API server
- Node is temporarily marked as not ready, blocking Pods being scheduled
- GC and Evictions are paused for the time of kuebelt restart and some graceful period. Causing slower reaction on memory pressure situation
- Ongoing image pulls are being canceled and depending on container runtime may cause a full re-pull.
- Pods admission will re-run.
  - If labels and tolerations set changed on the node, pods may fail the admission. TODO: find an issue in kubernetes repository describing this behavior

Overall in a healthy cluster, kubelet restart will not break anything. However in large clusters with overcommitted nodes, system instability can be caused by this restart.

2. Impact of container runtime restart
- All exec probes are failing for the duration of restart. This may cause Pod restarts (liveness probes with short timeouts and failure threshold) or Ready state flakiness.
- Node is marked as not ready by the kubelet. This blocks pods scheduling on the node.
- Some operations interruption historically caused state inconsistency, but those are edge cases
  - Interrupted image pull MAY create inconsistent image layers - rendering the image unusable (we had reports about it, but never a clear repro).
  - Interrupted sandbox creation (if terminated in the middle of CNI or NRI call), may create a sandbox in inconsistent state with CNI half-initialized and some resource leak may happen (we invested into improving this, but there is still a small chance for it to happen).
- Init containers execution stage may get lost and those init containers will re-run
- Container operations (restarts, initialization, etc.) will be delayed for the duration of restart.

Overall, to cause issues with the containerd restart, it should happen in a very precise moment, which is a low probability situaiton. So generally it is a safe operation. However, in case of heavy-loaded node when all processes are slow, the probability of interrupting some critical operation increases. 

3. Impact of a node reboot

TODO: describe what happens
- Device Plugin will be called to confirm devices allocation for the Pod
