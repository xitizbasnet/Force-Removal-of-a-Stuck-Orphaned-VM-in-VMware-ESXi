# 🧹 Force Removal of a Stuck / Orphaned VM in VMware ESXi

### *A Practical Troubleshooting Guide for Unresponsive VMs*

This documentation outlines the step-by-step procedure used to remove an unresponsive or orphaned virtual machine (VM) from an ESXi host when it cannot be powered off or deleted from the UI.

***

## 📘 Overview

In some cases, a VM may become unresponsive due to improper shutdown, corrupted state, storage issues, or management agent failures.  
When this happens, ESXi refuses to power it off or delete it through the vSphere/ESXi interface.

This manual provides a reliable, CLI-based method to:

✔ Identify the stuck VM  
✔ Attempt a graceful shutdown  
✔ Force-stop leftover VM processes  
✔ Unregister the VM from ESXi  
✔ Refresh host services  
✔ Validate successful cleanup

***

# 🔧 Procedure

***

## 1. 📄 List All Registered VMs

Identify the **VMID** of the problematic VM.

```bash
vim-cmd vmsvc/getallvms
```

From the output, note the VMID (example used here: `17`).

***

## 2. 📴 Attempt a Graceful Power-Off

```bash
vim-cmd vmsvc/power.off 17
```

If the VM remains powered or hung, continue to the next steps.

***

## 3. 🧠 Check Active VM Processes

Sometimes a low-level VM process is still running.

```bash
esxcli vm process list
```

If the VM appears here, you can force-kill it using `esxcli vm process kill` (optional and dangerous; not shown here unless needed).

***

## 4. 📂 Verify VM Folder on Datastore

Check whether the VM files exist.

```bash
ls -l /vmfs/volumes/datastore100/TestDeleteSRV/
```

This helps confirm the datastore path and pre-check for cleanup.

***

## 5. 🗑 Unregister VM from ESXi Inventory

This removes the VM entry from ESXi — it does **not** delete files.

```bash
vim-cmd vmsvc/unregister 17
```

If the VM is corrupted or orphaned, this step usually succeeds even when power-off fails.

***

## 6. 🔄 Restart ESXi Management Services

This refreshes **hostd**, **vpxa**, and associated services.

```bash
services.sh restart
```

⚠️ **Note:**  
This causes a temporary disconnect from vCenter. Running VMs are *not affected*.

***

## 7. 🧪 Re-check VM State / VMID

After restarting services, verify if the VM still exists.

```bash
vim-cmd vmsvc/power.getstate 17
```

If the VMID does not exist or returns an error — the VM is successfully removed.

***

# ✅ Final Result

The unresponsive VM was:

*   Powered off (forcefully, if needed)
*   Unregistered from the ESXi host
*   Validated through refreshed ESXi services

This method is safe, effective, and commonly used for resolving inventory corruption or orphaned VMs in VMware ESXi.

***
