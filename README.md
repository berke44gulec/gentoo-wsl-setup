# Gentoo on WSL: Optimized, Automated, and Documented

![Gentoo WSL Fastfetch](screenshots/fastfetch.png)

This repository serves as a comprehensive guide and configuration source for deploying a highly optimized Gentoo Linux environment on Windows Subsystem for Linux (WSL2).

It includes my personal configurations for a resource-efficient workspace, Ansible playbooks for automated setup, and a troubleshooting log for overcoming common WSL-specific challenges.

## Key Features

* **Optimized Compilation:** `make.conf` configured with `march=native` and specific `CPU_FLAGS_X86` for maximum hardware utilization.
* **Infrastructure as Code (IaC):** System setup and package management automated via **Ansible**.
* **WSL Tuning:** `.wslconfig` tailored for balanced resource usage between Windows and Linux.
* **Battle-Tested Solutions:** A log of real-world installation errors and their fixes.

## Repository Structure

```text
.
├── ansible/
│   └── playbook.yml    # Automation script for packages and users
├── configs/
│   ├── make.conf       # Portage compiler flags and USE settings
│   └── .wslconfig      # WSL resource limits (RAM/CPU)
├── screenshots/
└── README.md

Configuration & Optimization
1. WSL Resource Limits (.wslconfig)
To prevent the Linux VM from consuming all host RAM, copy the provided configuration file to your Windows user directory:
 * Source: configs/.wslconfig
 * Destination: C:\Users\<YourUser>\.wslconfig
2. Portage Optimization (make.conf)
The configs/make.conf file includes flags for a headless, optimized environment.
> ⚠️ CRITICAL WARNING:
> The CPU_FLAGS_X86 line in my make.conf is specific to my hardware.
> DO NOT use this line blindly on a different machine. You must generate your own flags:
>  * Install the tool: emerge app-portage/cpuid2cpuflags
>  * Run: cpuid2cpuflags
>  * Replace the CPU_FLAGS_X86 line in your config with the output.
> 
Automation (Ansible)
Instead of manually installing packages, you can use the provided Ansible playbook to set up the environment, install essential tools (git, vim, strace, etc.), and configure sudo access.
# 1. Install Ansible
emerge app-admin/ansible

# 2. Run the playbook
cd ansible
ansible-playbook playbook.yml

Troubleshooting Log
1. PowerShell curl Alias Issue
Error: Invoke-WebRequest : A parameter cannot be found that matches parameter name 'L'.
Solution: Use curl.exe explicitly.
Invoke-WebRequest -Uri "[https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-openrc/stage3-amd64-openrc-20251012T210603Z.tar.xz](https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-openrc/stage3-amd64-openrc-20251012T210603Z.tar.xz)" -OutFile C:\wsl\gentoo\stage3.tar.xz

2. Package Category Confusion
Corrections for common tools encountered during setup:
 * Git: Use dev-vcs/git (not sys-devel/git)
 * Strace: Use dev-debug/strace (not dev-util/strace)
3. Sudo Configuration
After installing sudo, you must manually enable the wheel group:
 * Run visudo
 * Uncomment: %wheel ALL=(ALL:ALL) ALL
Post-Installation Cleanup
To maintain a small WSL disk image size, remove unused dependencies after installation:
emerge -av --depclean
eclean-dist

Setting Default User
By default, imported WSL distributions log in as root. To configure WSL to use your standard user account:
Get-Item "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss\*\DistributionName" | Where-Object { $_.GetValue("") -eq "Gentoo" } | Set-ItemProperty -Name "DefaultUid" -Value 1000

License
This project is licensed under the MIT License.

