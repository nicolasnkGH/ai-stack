# Installation Guide
This document outlines the deployment of the Private AI Stack on an Ubuntu 24.04 VM. It assumes a functional Proxmox VE environment is already in place.

---

## 1. Prerequisites & Hardware Passthrough

**Summary:** Before deploying the VM, configure Proxmox for IOMMU and PCIe passthrough.

Before VM deployment, the Proxmox host must be configured for IOMMU and PCIe passthrough.
*   **IOMMU Configuration:** Ensure `intel_iommu=on` or `amd_iommu=on` is enabled in the host GRUB configuration.
    
*   **GPU Passthrough:** The NVIDIA RTX 3090 Ti must be mapped to the VM. Reference the official Proxmox [PCI Passthrough](https://pve.proxmox.com/wiki/PCI_Passthrough) for VFIO-PCI binding and IOMMU grouping.
    
*   **CPU Type:** Set the VM CPU type to `host` in Proxmox settings to enable AVX2/AVX-512 instructions required for LLM performance.

## 2. Persistent Storage (NFS & ZFS)

**Summary:** Configure persistent storage using an external ZFS-backed NAS.

The stack uses an external ZFS-backed NAS for data persistence.
1.  **Mount Point:** Create the local directory: `sudo mkdir -p /mnt/ai`.
    
2.  **Fstab Configuration:** Add the following line to `/etc/fstab` to ensure the NAS mount survives reboots:
    
    ```plaintext
    <NAS_IP>:/mnt/pool/ai-data /mnt/ai nfs defaults,soft,intr,noatime,rsize=1048576,wsize=1048576 0 0
    ```
    
    _Note:_ `rsize/wsize` are tuned to 1M to match the ZFS recordsize for large model blobs.
    
3.  **Apply Mount:** `sudo mount -a`.

## 3. Host Dependencies (Native)

**Summary:** Install necessary dependencies on the Ubuntu VM.

Install NVIDIA drivers and the Ollama runtime directly on the Ubuntu VM.
1.  **NVIDIA Drivers:** Install the production-branch drivers:
    
    ```bash
    sudo apt update && sudo apt install -y nvidia-driver-550 nvidia-utils-550
    ```
    
2.  **Ollama:** Install the native service for optimal GPU communication:
    
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    ```
    
3.  **Zabbix Agent 2:** Install the agent for external observability:
    
    ```bash
    sudo apt install zabbix-agent2
    sudo systemctl enable --now zabbix-agent2
    ```

## 4. Docker Stack Deployment

**Summary:** Deploy secondary services using Docker Compose.

All secondary services (Open WebUI, ComfyUI, etc.) are managed via Docker Compose.
1.  **Install Docker & Toolkit:** Ensure the `nvidia-container-toolkit` is installed so containers can access the 3090 Ti.
    
2.  **Initialize Stack:**
    
    ```bash
    cd ai-stack/config
    docker compose up -d
    ```

## 5. Verification

**Summary:** Verify proper configuration and connectivity.

*   **GPU:** Run `nvidia-smi` to confirm driver communication and VRAM availability.
    
*   **LLM:** Run `ollama list` to verify native models are accessible.
    
*   **Connectivity:** Access the Open WebUI via the Cloudflare Tunnel URL and verify the Zabbix server is receiving active checks from Agent 2.

---

## 🔗 References & External Resources

- [Proxmox PCI Passthrough Guide](https://pve.proxmox.com/wiki/PCI_Passthrough) - Official documentation for host-side IOMMU and VFIO setup.
- [NVIDIA Container Toolkit Documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) - Detailed installation steps for GPU-enabled Docker.
- [Ollama Linux Manual](https://github.com/ollama/ollama/blob/main/docs/linux.md) - Configuration details for the native Ollama service.
- [Zabbix Agent 2 Documentation](https://www.zabbix.com/documentation/current/en/manual/concepts/agent2) - Advanced configuration for active/passive checks.

