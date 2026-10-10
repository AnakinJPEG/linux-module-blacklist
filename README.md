# Disabling a Hardware Bus via Kernel Module Blacklisting

A targeted Linux system fix that resolves a hardware conflict and reduces attack surface by preventing an unused bus module from loading at boot.

## The Problem

A hardware conflict was causing device errors and instability on the system. An onboard bus interface was loading a kernel module that conflicted with other hardware, producing errors in `dmesg` and preventing the system from behaving reliably.

The interface was not in use, but the kernel was still initializing it at boot.

## The Solution

Blacklist the offending kernel module so it never loads, eliminating the conflict at the source without disabling unrelated functionality.

## Diagnosis

**1. Identify the errors:**
```bash
dmesg | grep -i error
```

**2. Identify the module bound to the device:**
```bash
lspci -k
```
Look for the `Kernel driver in use:` line under the relevant device.

**3. Confirm the module is loaded:**
```bash
lsmod | grep <module_name>
```

## The Fix

**1. Create a blacklist file:**
```bash
sudo nano /etc/modprobe.d/blacklist-<module>.conf
```

**2. Add the following:**
```
blacklist <module_name>
```

**3. Rebuild the initramfs (so the module is excluded at early boot):**
```bash
sudo update-initramfs -u
```
> On RHEL/Fedora-based systems: `sudo dracut -f`

**4. Reboot:**
```bash
sudo reboot
```

## Verification

After reboot, confirm the module is no longer loaded:
```bash
lsmod | grep <module_name>
```
No output = success.

Check that the original errors are gone:
```bash
dmesg | grep -i error
```

## Why This Works

Blacklisting prevents the kernel from auto-loading the module. Rebuilding the initramfs ensures it is also excluded from the early boot environment, so it never initializes even before the root filesystem is mounted.

## What I Learned

- How the Linux kernel binds drivers to hardware via `lspci -k` and `lsmod`
- The role of `modprobe.d` and initramfs in module loading
- How disabling unused hardware reduces both **conflict risk** and **attack surface**
- The importance of verifying fixes with `dmesg` and `lsmod`

## ⚠️ Cautions

- Only blacklist modules you are certain are unused. Disabling a storage or network driver can render a system unbootable.
- Always keep a live USB or recovery method available before rebooting.
- Document changes so they can be reversed.

## Reverting the Change

Remove or comment out the blacklist entry, then rebuild initramfs and reboot:
```bash
sudo rm /etc/modprobe.d/blacklist-<module>.conf
sudo update-initramfs -u
sudo reboot
```

## Requirements

- Linux system with `modprobe`, `dmesg`, `lspci`
- Root/sudo access
- A recovery method in case of boot issues

## License

MIT
