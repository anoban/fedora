## ___Installing `NVIDIA` proprietary graphics drivers on Fedora 44___
----------------------------------

1. Download the [latest driver](https://www.nvidia.com/en-us/drivers/details/267258/) from `NVIDIA`'s official website, that supports the `GeForce MX330` card this machine has. As of `06-05-2026`, the latest available version is `580.159.03`.

2. Give the installer execution privileges `$ chmod +x ./NVIDIA-Linux-x86_64-580.159.03.run`.

3. Then run the installer with superuser privileges `$ sudo ./NVIDIA-Linux-x86_64-580.159.03.run`.

4. During the installation, the driver's kernel module will need signing on `secure boot` enabled machines that use `UEFI`. For this, allow the installer to generate a new private and public key pair and use it to sign the kernel module. Run `$ mokutil --sb-state` to check if secure boot is enabled. If it is, the installer will automatically recognize it and notify you.

5. The storage location of these keys will be informed by the installer, post signing. But is typically `/usr/share/nvidia/`.

    ```
    /usr/share/nvidia/nvidia-modsign-crt-DC8577D7.der - public key
    /usr/share/nvidia/nvidia-modsign-key-DC8577D7.key - private key
    ```

6. The public key needs to be added to a database trusted by the OS kernel, so the NVIDIA driver's kernel module loads successfully at boot time. The installer will explicitly advise you about this.

7. Post signing, if the system has the Dynamic Kernel Module System - `DKMS` installed, the installer will prompt you to register the driver's kernel module with `DKMS`, so whenever the kernel gets an update, the driver's kernel module will be rebuilt against the newer kernel, automatically. Do so if desired.

8. If the system had a `nouveau` driver prior to the installation of the `NVIDIA` driver, the installer will prompt you to rebuild `initramfs`.

9. After exiting the installer, enroll the public key used to sign the driver's kernel module into the Machine Owner Key - `MOK` database. So that the kernel knows that the sgnature on the module is trusted. [RPMFusion docs](https://rpmfusion.org/Howto/Secure%20Boot) To do this run:

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
