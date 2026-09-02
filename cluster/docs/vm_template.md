# VM Resource Requirements

Before creating VMs, familiarise yourself with the system requirements for Talos and use the following as a baseline for all Talos nodes on Proxmox:

| Setting | Recommended value | Notes |
| :--- | :--- | :--- |
| **BIOS** | `ovmf` (UEFI) | Modern firmware, Secure Boot support, better hardware compatibility |
| **Machine** | `q35` | Modern PCIe-based machine type with better device support |
| **CPU type** | `host` | Enables advanced instruction sets (AVX-512, etc.). Use `kvm64` with feature flags for Proxmox < 8.0 |
| **CPU cores** | 2+ (control plane), 4+ (workers) | Minimum 2 cores required |
| **Memory** | 4GB+ (control plane), 8GB+ (workers) | Minimum 2GB required |
| **Disk controller** | VirtIO SCSI | **Do NOT use VirtIO SCSI Single** — causes bootstrap hangs (#11173) |
| **Disk format** | Raw or QCOW2 | Raw preferred for performance; QCOW2 for snapshots |
| **Disk cache** | Write Through | Use `None` for clustered environments |
| **Network model** | virtio | Paravirtualized driver, best performance (up to 10 Gbit) |
| **EFI disk** | 4MB | Required for UEFI firmware, stores Secure Boot keys |
| **Ballooning** | Disabled | Talos does not support memory hotplug |
| **RNG device** | VirtIO RNG *(optional)* | Better entropy for cryptographic operations |