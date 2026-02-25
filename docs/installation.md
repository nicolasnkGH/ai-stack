# Installation Guide
This document outlines the deployment of the Private AI Stack on an Ubuntu 24.04 VM. It assumes a functional Proxmox VE environment is already in place.

---

## 1. Prerequisites & Hardware Passthrough

**Summary:** Before deploying the VM, configure Proxmox for IOMMU and PCIe passthrough.

Before VM deployment, the Proxmox host must be configured for IOMMU and PCIe passthrough.
*   **IOMMU Configuration:** Ensure `intel_iommu=on` or `amd_iommu=on` is enabled in the host GRUB configuration.
    
*   **GPU Passthrough:** The NVIDIA RTX 3090 Ti must be mapped to the VM. Reference the official Proxmox [PCI Passthrough](https://pve.proxmox.com/wiki/PCI_Passthrough) for VFIO-PCI binding and IOMMU grouping.
    
*   **CPU Type:** Set the VM CPU type to `host` in Proxmox settings to enable AVX2/AVX-512 instructions required for LLM performance.

## 2. Persistent Storage Options

**Summary:** You can choose between a performance-optimized hybrid model or a simplified local-only setup.

### Option A: Hybrid Storage Model (Recommended for Scaling)

This is the architecture used in this repository, splitting data between high-speed local SSD and high-capacity NAS.

1.  **Local "Hot" Storage (SSD)**: Create directories for performance-critical data:

    ```bash
    sudo mkdir -p /opt/ai/{ollama,comfyui/models,open-webui,redis}
    sudo chown -R 1000:1000 /opt/ai
    ```
---

2.  **Bulk "Cold" Storage (NFS/ZFS)**: Mount your NAS for large assets:

    *   **Mount Point**: `sudo mkdir -p /mnt/ai`.

    *   **Fstab Entry**: `<NAS_IP>:/mnt/pool/ai-data /mnt/ai nfs defaults,soft,intr,noatime,rsize=1048576,wsize=1048576 0 0`.

### Option B: Local-Only Storage (Simplified)

If you prefer not to use a NAS, you can store everything on the VM's local disk.

*   **Setup**: Simply map all volumes in `docker-compose.yml` to paths within `/opt/ai` or your home directory.

*   **Trade-off**: High performance, but limited by the virtual disk size allocated in Proxmox.
    
3.  **Apply Mount:** `sudo mount -a`.

---

## 3. Host Dependencies

**Summary:** Install necessary drivers and the NVIDIA container toolkit on the Ubuntu VM.

1.  **NVIDIA Drivers**: Install the production-branch drivers (e.g., 550 or newer):

    ```bash
    sudo apt update && sudo apt install -y nvidia-driver-550 nvidia-utils-550
    ```

2.  **NVIDIA Container Toolkit**: **Mandatory.** Enables the containerized Ollama and ComfyUI to utilize the 3090 Ti.

3.  **Zabbix Agent 2**: Install for out-of-band infrastructure monitoring.

---

## 4. Docker Stack Deployment

**Summary:** Deploy the full stack (including the LLM engine) via Docker Compose.

1.  **Environment Setup**: Initialize your secrets file:

    ```bash
    cp .env.example .env
    # Edit .env to add your optional Google Drive API keys
    ```

2.  **Launch**:

    ```bash
    cd ai-stack/
    docker compose up -d
    ```

3.  **Verification**: After a few minutes, verify all services are running with:

    ```bash
    docker compose ps
    ```

    **Expected Output:**

    ```
    NAME           IMAGE                                      COMMAND                  SERVICE      CREATED       STATUS                 PORTS
    ai-comfyui     yanwk/comfyui-boot:cu124-slim              "bash -c 'pip instal…"   comfyui      2 hours ago   Up 2 hours             0.0.0.0:8188->8188/tcp, [::]:8188->8188/tcp
    ai-ollama      ollama/ollama:latest                       "/bin/ollama serve"      ollama       2 hours ago   Up 2 hours             0.0.0.0:11434->11434/tcp, [::]:11434->11434/tcp
    ai-openwebui   ghcr.io/open-webui/open-webui:main         "bash start.sh"          open-webui   2 hours ago   Up 2 hours (healthy)   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp
    ai-redis       redis:7-alpine                             "docker-entrypoint.s…"   redis        2 hours ago   Up 2 hours             6379/tcp
    ai-searxng     searxng/searxng:latest                     "/usr/local/searxng/…"   searxng      2 hours ago   Up 2 hours             0.0.0.0:8088->8080/tcp, [::]:8088->8080/tcp
    ai-tts         ghcr.io/matatonic/openedai-speech:latest   "/bin/sh -c 'bash st…"   tts          2 hours ago   Up 2 hours             0.0.0.0:8001->8000/tcp, [::]:8001->8000/tcp
    ```


---

## 5. Maintenance & Verification

**Summary:** Configure automated backups and verify GPU acceleration.

1.  **Database Backups**: Protect your local SQLite DB by syncing it to your backup location nightly:

    ```bash
    chmod +x ./scripts/backup-db.sh
    (crontab -l 2>/dev/null; echo "0 3 * * * /home/ubuntu/ai-stack/scripts/backup-db.sh") | crontab -
    ```

2.  **Verification**: Confirm `nvidia-smi` shows active processes for both `ollama` and `comfyui`.


---

## 🔗 References & External Resources

- [Proxmox PCI Passthrough Guide](https://pve.proxmox.com/wiki/PCI_Passthrough) - Official documentation for host-side IOMMU and VFIO setup.
- [NVIDIA Container Toolkit Documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) - Detailed installation steps for GPU-enabled Docker.
- [Ollama Linux Manual](https://github.com/ollama/ollama/blob/main/docs/linux.md) - Configuration details for the native Ollama service.
- [Zabbix Agent 2 Documentation](https://www.zabbix.com/documentation/current/en/manual/concepts/agent2) - Advanced configuration for active/passive checks.

---
_Maintained by Nicolas Teixeira._

