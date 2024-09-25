# nvidia-fedora-setup

## Installing and Signing Nvidia Drivers for RTX 4090 on Fedora with Secure Boot

## Prerequisites
Ensure your system is up to date:
```bash
sudo dnf update -y
```

## Step 1: Enable RPM Fusion Repositories
Nvidia drivers are available from the RPM Fusion repositories. Enable both the free and non-free repositories:
```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

## Step 2: Install Nvidia Drivers
Install the appropriate Nvidia drivers for your RTX 4090. The `akmod-nvidia` package is recommended, as it automatically builds kernel modules when the kernel is updated.
```bash
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda
```

This will install the Nvidia driver, CUDA support, and the necessary tools.

### Check Kernel Headers and Dependencies
Make sure you have the kernel headers and development tools installed to build the kernel modules:
```bash
sudo dnf install kernel-headers kernel-devel gcc
```

### Optional: Verify Driver Installation
To verify that the Nvidia driver is correctly installed:
```bash
nvidia-smi
```

This command should display information about your Nvidia GPU (e.g., RTX 4090).

## Step 3: Reboot the System
Reboot to allow the changes to take effect:
```bash
sudo reboot
```

## Step 4: Secure Boot Issues (Nvidia Drivers Won't Load)
If Secure Boot is enabled, the Nvidia driver modules will not load because they are unsigned. To solve this, we need to sign the Nvidia kernel module.

## Step 5: Install Required Tools for Signing
Install `mokutil` and `openssl` to create signing keys and sign the kernel modules:
```bash
sudo dnf install mokutil openssl
```

## Step 6: Generate a Signing Key
Create a directory for storing your signing keys and generate a key pair:
```bash
mkdir ~/kernel-signing
cd ~/kernel-signing
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Kernel Module Signing/"
```

## Step 7: Find the Nvidia Kernel Module Path
Locate the Nvidia kernel module installed on your system:
```bash
find /lib/modules/$(uname -r) -name "nvidia.ko"
```

In this case, the correct path is likely:
```
/lib/modules/$(uname -r)/kernel/drivers/video/nvidia.ko
```

## Step 8: Sign the Nvidia Kernel Module
Use the generated key to sign the Nvidia kernel module:
```bash
/usr/src/kernels/$(uname -r)/scripts/sign-file sha256 /home/sln/kernel-signing/MOK.priv /home/sln/kernel-signing/MOK.der /lib/modules/$(uname -r)/kernel/drivers/video/nvidia.ko
```

## Step 9: Enroll the Key with Secure Boot
Now, import the signing key to the system's MOK (Machine Owner Key) list so Secure Boot will trust it:
```bash
sudo mokutil --import /home/sln/kernel-signing/MOK.der
```

You'll be prompted to set a password. This is required to enroll the key during the next boot.

## Step 10: Reboot and Enroll the MOK Key
Reboot your system. During the boot process, you’ll be prompted to enroll the MOK key. Follow the on-screen instructions and provide the password you created earlier.

Once the key is enrolled, the Nvidia driver should load without issue, even with Secure Boot enabled.

## Step 11: Verify the Nvidia Driver is Loaded
After rebooting, verify that the Nvidia kernel module is loaded properly:
```bash
sudo lsmod | grep nvidia
```

You can also check if the Nvidia GPU is recognized:
```bash
nvidia-smi
```

### Optional: Verify the Module is Signed
You can verify if the Nvidia kernel module was signed successfully:
```bash
sudo modinfo /lib/modules/$(uname -r)/kernel/drivers/video/nvidia.ko | grep "sig"
```

## Troubleshooting
- **Nvidia Module Not Found:** If the Nvidia module doesn't load, ensure you have installed `akmod-nvidia` and the appropriate kernel headers.
- **Secure Boot Fallback:** If Secure Boot continues to block the module, ensure you have enrolled the correct MOK key and signed the appropriate Nvidia kernel module.

## Summary:
By following these steps, you have successfully installed the Nvidia drivers, signed the Nvidia kernel module, and enrolled the signing key to ensure the drivers work with Secure Boot enabled.
```

This guide covers installing Nvidia drivers from scratch and signing the kernel module to work with Secure Boot enabled on Fedora. Let me know if you need further clarification or adjustments!
