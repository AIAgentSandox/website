# explain allocatable drift

## Implementation Steps

### Task 1: explain allocatable drift

- [x] Today `--kube-reserved` and `--system-reserved` are subtracted from node **capacity** to derive **allocatable**. However, the "capacity" kubelet reports is **not raw hardware capacity** — it is `MemTotal` from `/proc/meminfo`, which the kernel computes after subtracting its own reservations (kernel image, `vmemmap`, `crashkernel`, `initrd`, ACPI tables, firmware-reserved regions, hugepages set at boot, MMIO holes, etc.). When the OS is patched and the new kernel reserves a different amount, **capacity** drifts and therefore **allocatable** drifts even though the operator's `kube-reserved`/`system-reserved` numbers are unchanged.

Find the page where this observation will be best suited for and add it.
