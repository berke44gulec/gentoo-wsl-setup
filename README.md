# Gentoo on WSL: Installation Notes and Troubleshooting

This repository documents the process of installing Gentoo Linux on Windows Subsystem for Linux (WSL). It specifically focuses on real-world errors encountered during the installation and their solutions, serving as a practical supplement to the official Gentoo Wiki.

## Prerequisites

* Windows 10 or 11
* WSL2 enabled
* Windows Terminal

## Note
It is recommended that you use current versions and use the updated download links at https://www.gentoo.org/downloads .

## Troubleshooting Log

### 1. PowerShell curl Alias Issue

When attempting to download the Stage3 tarball using PowerShell, the standard `curl` command fails because PowerShell aliases `curl` to `Invoke-WebRequest`, which does not support the same flags (specifically `-L`).

**Error:**
`Invoke-WebRequest : A parameter cannot be found that matches parameter name 'L'.`

**Solution:**
Use the native PowerShell syntax or explicitly call `curl.exe`.

```powershell
# Correct PowerShell command:
Invoke-WebRequest -Uri "[https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-openrc/stage3-amd64-openrc-20251012T210603Z.tar.xz](https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-openrc/stage3-amd64-openrc-20251012T210603Z.tar.xz)" -OutFile C:\wsl\gentoo\stage3.tar.xz
