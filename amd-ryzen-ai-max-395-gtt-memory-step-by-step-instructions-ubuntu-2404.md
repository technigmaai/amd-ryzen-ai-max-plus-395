# AMD Ryzen AI Max+ 395: GTT  Memory Step-by-Step Instructions (Ubuntu 24.04)

## **1. Set VRAM to 512MB and Disable IOMMU in BIOS**


1. Reboot your computer and enter the BIOS/UEFI setup (usually via Del, F2, or Esc during startup).
2. Find the setting for **"Integrated Graphics"** or **"UMA Frame Buffer Size"** (name varies by vendor).
3. Set VRAM size to **512MB**.
4. **Disable** IOMMU
5. Save and exit.


---

## **2. Edit GRUB to Add Kernel Boot Options**

Open a terminal and run:

```bash
sudo nano /etc/default/grub
```

Find the line:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

Modify it to include:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash amd_iommu=off amdgpu.gttsize=131072 ttm.pages_limit=335544321"
```



:::info
Explanation of parameters:

* amd_iommu=off – disables AMD IOMMU to reduce latency
* amdgpu.gttsize=131072 – increases GTT memory size to 128GB
* ttm.pages_limit=335544321 – sets the page limit to support large memory pool

:::


---

## **3. Update GRUB and Reboot**

Run:

```bash
sudo update-grub
```

Then reboot:

```bash
sudo reboot
```


---

## **4. Verify Settings After Boot**

After reboot, verify kernel params were applied:

```bash
cat /proc/cmdline
```

You should see the flags you added (e.g. amd_iommu=off, amdgpu.gttsize=131072).

To verify GTT memory usage and TTM page limit:

```bash
sudo dmesg | grep -i gtt
```

And for TTM:

```bash
sudo dmesg | grep -i ttm
```

You may also inspect the AMD GPU driver settings via:

```bash
sudo cat /sys/module/amdgpu/parameters/gttsize
sudo cat /sys/module/amdgpu/parameters/ttm_pages_limit # I don't have in in Ubuntu 24.04
```


---

## **5. You're Done!**

You should now have **access to full 128GB of system memory** for your GPU via GTT, while keeping it shared with the CPU when needed — no more 96GB ceiling.