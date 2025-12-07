# Gentoo on WSL2: Production Configuration and Automation

![Gentoo WSL Fastfetch](screenshots/fastfetch.png)

This repository provides a complete configuration framework for deploying an optimized Gentoo Linux environment on Windows Subsystem for Linux 2 (WSL2). It includes production-grade compiler optimizations, automated provisioning via Ansible, and documented solutions to common deployment challenges.

## Features

- **Hardware-Optimized Compilation:** Portage configuration utilizing `march=native` and processor-specific `CPU_FLAGS_X86` for optimal binary performance
- **Automated Provisioning:** Complete system deployment through Ansible playbooks
- **Resource Management:** WSL configuration tuned for efficient host resource allocation
- **Production Documentation:** Comprehensive troubleshooting reference based on real deployment scenarios

## Repository Structure

```text
.
├── ansible/
│   └── playbook.yml    # System provisioning automation
├── configs/
│   ├── make.conf       # Portage compiler configuration
│   └── .wslconfig      # WSL resource allocation limits
├── screenshots/
└── README.md
```

## Prerequisites

Before using this configuration, ensure you have:

- WSL2 enabled on Windows 10/11
- A Gentoo stage3 tarball imported into WSL
- Basic familiarity with Gentoo's package management (Portage)

## Configuration

### WSL Resource Allocation

Copy the provided WSL configuration to your Windows user directory:

- **Source:** `configs/.wslconfig`
- **Destination:** `C:\Users\<YourUsername>\.wslconfig`

The provided configuration allocates:
- **5 CPU cores**
- **3 GB RAM**
- **20 GB virtual hard disk size**

Adjust these values based on your hardware specifications and requirements.

After copying the file, restart WSL:

```powershell
wsl --shutdown
```

### Portage Compiler Optimization

Copy `configs/make.conf` to `/etc/portage/make.conf` in your Gentoo WSL instance.

> **⚠️ CRITICAL: Hardware-Specific Configuration**
> 
> The `CPU_FLAGS_X86` line in the provided `make.conf` is generated for specific hardware and **must not be used directly** on different systems. This configuration is hardware-dependent and using incorrect flags can cause package compilation failures or runtime errors.
> 
> **Required steps for your system:**
> 
> 1. Install the CPU flags detection tool:
>    ```bash
>    emerge app-portage/cpuid2cpuflags
>    ```
> 
> 2. Generate your hardware-specific flags:
>    ```bash
>    cpuid2cpuflags
>    ```
> 
> 3. Replace the `CPU_FLAGS_X86` line in `/etc/portage/make.conf` with the generated output.

**Configuration Highlights:**

- **MAKEOPTS="-j5"**: Parallel compilation using 5 jobs (adjust based on your CPU core count)
- **USE flags**: Minimal headless configuration with desktop environments and unnecessary services disabled
- **ACCEPT_LICENSE="*"**: Accepts all software licenses (review based on your requirements)

## Automated Deployment

### Installing Ansible

First, install Ansible in your Gentoo WSL environment:

```bash
emerge --ask app-admin/ansible
```

### Running the Playbook

The Ansible playbook automates:
- Portage tree synchronization
- Essential package installation (sudo, vim, git, strace, tcpdump)
- Sudo configuration for the wheel group
- Dependency cleanup

Execute the playbook:

```bash
cd ansible
ansible-playbook playbook.yml
```

**Note:** The playbook requires root privileges and will prompt for confirmation during package installation.

### Manual Alternative

If you prefer manual installation without Ansible:

```bash
# Sync portage tree
emerge --sync

# Install packages
emerge --ask app-admin/sudo app-editors/vim dev-vcs/git dev-debug/strace net-analyzer/tcpdump

# Configure sudo
visudo
# Uncomment: %wheel ALL=(ALL:ALL) ALL

# Clean dependencies
emerge --ask --depclean
```

## Troubleshooting

### PowerShell curl Alias Conflict

**Error:** `Invoke-WebRequest : A parameter cannot be found that matches parameter name 'L'.`

**Solution:** PowerShell aliases `curl` to `Invoke-WebRequest`. Use the full cmdlet or the native curl executable:

```powershell
Invoke-WebRequest -Uri "https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-openrc/stage3-amd64-openrc-20251012T210603Z.tar.xz" -OutFile C:\wsl\gentoo\stage3.tar.xz
```

### Package Category Reference

Correct package atoms for commonly used utilities:

- **Git:** `dev-vcs/git` (not `sys-devel/git`)
- **Strace:** `dev-debug/strace` (not `dev-util/strace`)
- **Sudo:** `app-admin/sudo`

### Sudo Wheel Group Configuration

If the Ansible playbook fails to configure sudo, manually enable wheel group access:

```bash
visudo
```

Uncomment the following line:

```
%wheel ALL=(ALL:ALL) ALL
```

### Compilation Failures

If packages fail to compile after applying `make.conf`:

1. Verify your `CPU_FLAGS_X86` is correctly generated for your hardware
2. Test with safer flags temporarily:
   ```bash
   COMMON_FLAGS="-O2 -pipe -march=x86-64"
   ```
3. Check `/var/log/portage/` for specific error messages

## Post-Installation Maintenance

### Dependency Cleanup

Remove unnecessary dependencies and cached distribution files:

```bash
emerge --ask --depclean
eclean-dist
```

### Regular Updates

Keep your system updated:

```bash
emerge --sync
emerge --ask --update --deep --newuse @world
```

## Default User Configuration

WSL distributions import as root by default. Configure your standard user account as the default login:

```powershell
Get-Item "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss\*\DistributionName" | Where-Object { $_.GetValue("") -eq "Gentoo" } | Set-ItemProperty -Name "DefaultUid" -Value 1000
```

Replace `1000` with your user's UID if different (check with `id -u username` in WSL).

## License

This project is licensed under the MIT License.
