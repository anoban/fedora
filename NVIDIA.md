## ___Installing `NVIDIA` proprietary graphics drivers on Fedora 44, using `NVIDIA`'s official installers___
----------------------------------
</br>

1. Download the [latest driver](https://www.nvidia.com/en-us/drivers/details/267258/) from `NVIDIA`'s official website, that supports the `GeForce MX330` card this machine has. As of `06-05-2026`, the latest available version is `580.159.03`.

2. Give the installer execution privileges `$ chmod +x ./NVIDIA-Linux-x86_64-580.159.03.run`.

3. Then run the installer with superuser privileges `$ sudo ./NVIDIA-Linux-x86_64-580.159.03.run`.

4. During the installation, the driver's kernel module will need signing on `secure boot` enabled machines that use `UEFI`. For this, allow the installer to generate a new private and public key pair and use it to sign the kernel module. Run `$ mokutil --sb-state` to check if secure boot is enabled. If it indeed is, the installer will automatically recognize it and notify you. Alternatively, if you have a pair of private and public keys on your machine, you could choose to reuse them.

5. The storage location of the installer generated keys will be informed during the installation, post signing. But is typically `/usr/share/nvidia/`.

    ```
    /usr/share/nvidia/nvidia-modsign-crt-DC8577D7.der - public key
    /usr/share/nvidia/nvidia-modsign-key-DC8577D7.key - private key
    ```

6. The public key needs to be added to a database trusted by the OS kernel, so the `NVIDIA` driver's kernel module loads successfully at boot time. The installer will explicitly advise you about this.

7. Post signing, for installers of driver version later than `304.0`, if the system has the Dynamic Kernel Module System - `DKMS` installed, the installer will prompt you to register the driver's kernel module with `DKMS`, so whenever the kernel gets an update, the driver's kernel module will be rebuilt against the newer kernel, automatically. Do so if desired.

8. If the system had a `nouveau` driver prior to the installation of the `NVIDIA` driver, the installer will prompt you to rebuild `initramfs`.

9. After exiting the installer, enroll the public key used to sign the driver's kernel module into the Machine Owner Key - `MOK` database. So that the kernel knows that the sgnature on the module is trusted. [RPMFusion docs](https://rpmfusion.org/Howto/Secure%20Boot), [RedHat docs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/signing-a-kernel-and-modules-for-secure-boot_managing-monitoring-and-updating-the-kernel) To do this run (`sudo` is important here):

    ```
    $ sudo mokutil --import /usr/share/nvidia/nvidia-modsign-crt-DC8577D7.der
    ```

10. This enrollment will require a password which will be aksed again at reboot. Once the enrollment is complete, reboot the system `$ sudo reboot -f`.

11. During reboot, the `MOK` screen will appear, prompting you to enroll the key. Enroll the key, provide the password and reboot the machine. [Fedora docs](https://docs.fedoraproject.org/en-US/quick-docs/mok-enrollment/).

12. To verify successfull installation, run the following:

    `$ modinfo -F version nvidia` - should output the installed driver's version `580.159.03`.

    `$ sudo lspci -s 01:00.0 -v` - should ouput something like the following.

    ```
    01:00.0 3D controller: NVIDIA Corporation GP108M [GeForce MX330] (rev a1)
	Subsystem: Dell Device 0a25
	Flags: bus master, fast devsel, latency 0, IRQ 156
	Memory at 71000000 (32-bit, non-prefetchable) [size=16M]
	Memory at 6000000000 (64-bit, prefetchable) [size=256M]
	Memory at 6010000000 (64-bit, prefetchable) [size=32M]
	I/O ports at 4000 [size=128]
	Capabilities: [60] Power Management version 3
	Capabilities: [68] MSI: Enable+ Count=1/1 Maskable- 64bit+
	Capabilities: [78] Express Endpoint, IntMsgNum 0
	Capabilities: [100] Virtual Channel
	Capabilities: [250] Latency Tolerance Reporting
	Capabilities: [258] L1 PM Substates
	Capabilities: [128] Power Budgeting <?>
	Capabilities: [420] Advanced Error Reporting
	Capabilities: [600] Vendor Specific Information: ID=0001 Rev=1 Len=024 <?>
	Capabilities: [900] Secondary PCI Express
	Kernel driver in use: nvidia
	Kernel modules: nouveau, nvidia_drm, nvidia
    ```

    For a more verbose output, use `-vv` instead of `-v`. And the `-s` is the slot ID of the `NVIDIA` graphics card, which is machine dependent.
    Note the last two lines, suggesting the installed drivers are currently in use:
    ```
    Kernel driver in use: nvidia
	Kernel modules: nouveau, nvidia_drm, nvidia
    ```

    `$ nvidia-smi` - should output something like,
    ```
    Wed May  6 17:10:09 2026
    +-----------------------------------------------------------------------------------------+
    | NVIDIA-SMI 580.159.03             Driver Version: 580.159.03     CUDA Version: 13.0     |
    +-----------------------------------------+------------------------+----------------------+
    | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
    |                                         |                        |               MIG M. |
    |=========================================+========================+======================|
    |   0  NVIDIA GeForce MX330           Off |   00000000:01:00.0 Off |                  N/A |
    | N/A   50C    P8            N/A  / 5001W |       2MiB /   2048MiB |      0%      Default |
    |                                         |                        |                  N/A |
    +-----------------------------------------+------------------------+----------------------+

    +-----------------------------------------------------------------------------------------+
    | Processes:                                                                              |
    |  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
    |        ID   ID                                                               Usage      |
    |=========================================================================================|
    |    0   N/A  N/A            2866      G   /usr/bin/gnome-shell                      0MiB |
    +-----------------------------------------------------------------------------------------+
    ```


------------------------------


This method offers more granular control during installation compared to the RPMFusion based solutions.

PS. The private and public keys generated by the `NVIDIA` installer can be reused to sign other kernel modules since it has already been enrolled in the `MOK` database.

All the keys that are enrolled in the `MOK` database can be examined using `$ mokutil --list-enrolled`.


## ___Automatic kernel module rebuilds with `DKMS`, after kernel upgrades___
-------------------------------------
</br>

`DKMS` only helps with automatically rebuilding kernel modules when the kernel gets updated, it does not do anything to authenticate the kernel modules built against the newer kernel (i.e it does sign the rebuilt kernel modules but won't enroll the public key with `MOK`).

You can check the status of `DKMS` automated kernel module rebuilds with `$ dkms status`.

Hence, re enrollment of keys is needed when kernel modules get rebuilt by `DKMS` after a kernel upgrade.

The configs of `DKMS` can be found in the file `/etc/dkms/framework.conf`. This `framework.conf` file will contain the path to the public key used to sign the kernel module during the automatic rebuild, in variable `mok_certificate`, which is typically `/var/lib/dkms/mok.pub`.

Enroll this `DKMS` public key into `MOK` for the rebuilt kernel modules to work, after a kernel upgrade.

Logs of the automatic kernel module rebuild are usually stored in `/var/lib/dkms/nvidia/<driver-version>/<kernel-version>/<architecture>/log/`.

After a kernel upgrade, with successful `NVIDIA` kernel module rebuild, the system may still be unable to load the kernel modules if the certificate used to sign that module wasn't trusted in the `MOK` database. If this is the case, `nvidia-smi` would output `NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running`. If this happens, just enroll the `DKMS` public key in `MOK`, and reboot the system.


## ___Uninstall NVIDIA drivers installed from a `.run` file___
-----------------------------------


In certain instances, there may be issues rebuilding kernel modules for the existing NVIDIA driver, with the release of a newer Linux kernel version. This can happen due to the installed NVIDIA driver's kernel sources being incompatible or not up to date with the headers of the upgraded kernel headers. In that case, it's best to do a manual clean reinstall. First, uninstall the existing NVIDIA driver, this will require an NVIDIA installer `.run` file (this doesn't need to be the same version as the one already installed on the machine).

```
sudo NVIDIA-Linux-x86_64-580.173.02.run --uninstall
```

Will cleanup the existing driver installation. Then, do a manual install using the `.run` file, as described above.
