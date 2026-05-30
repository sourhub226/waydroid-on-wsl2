# Waydroid in WSL2 with sound

<details>
<summary>Prerequisite: Compiling custom linux kernel</summary>

> Waydroid needs a custom linux kernel.
> Actually just needs [latest kernel](https://github.com/microsoft/WSL2-Linux-Kernel/blob/linux-msft-wsl-6.6.y/arch/x86/configs/config-wsl) for WSL2 with just these changes before compiling:
>
>     CONFIG_ANDROID_BINDER_IPC=y
>     CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"

### Building x86_64 WSL2 6.6.x kernel with an Ubuntu distribution using **bash**:

- Install the build dependencies:<br>
  `sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev cpio qemu-utils libncurses-dev`

- Download kernel (always in a /home/user/directory)<br>
  `git clone https://github.com/microsoft/WSL2-Linux-Kernel.git --depth=1`

- Change directory to kernel's directory root<br>
  `cd WSL2-Linux-Kernel`

- Modify WSL2 kernel configs:<br>
  `make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl`

Device Drivers --> Android --> Select "Android Binder IPC Driver" -> Save

- Build the kernel using the WSL2 kernel configuration and put the modules in a modules folder under the current working directory:<br>
  `make -j$(nproc) KCONFIG_CONFIG=Microsoft/config-wsl && make INSTALL_MOD_PATH="$PWD/modules" modules_install`

- Strip symbols from modules (skip this step if you are insterested in debugging kernel's memory dumps)<br>
  `find ./modules/lib/modules/$(make -s kernelrelease) -name '*.ko' -exec strip --strip-unneeded {} \;`
- Then, you can use a provided script to create a VHDX containing the modules:<br>
  `sudo ./Microsoft/scripts/gen_modules_vhdx.sh "$PWD/modules" $(make -s kernelrelease) modules.vhdx`

- Copy the generated kernel and modules to the desired folder (D:\mykernel)<br>
`cp arch/x86/boot/bzImage /mnt/d/mykernel/bzImage-6.6.87.2.ANDROID`<br>
`cp modules.vhdx /mnt/d/mykernel/modules.ANDROID.vhdx`
</details>

## Setup

> If the prerequisite setup was successful, custom kernel will be in D:\mykernel

Open WSL Settings -> Developer -> Select Custom kernel, Custom modules

Shutdown WSL2 VM<br>
`wsl.exe --shutdown`

> It is recommended to install Waydroid in a brand new Ubuntu install.
>
> Download latest `ubuntu-XX-wsl-amd64.wsl` file from https://releases.ubuntu.com/ (for this guide distro Ubuntu 25.04 was used)
>
> Double click on the .wsl file to install.

Run Ubuntu and make sure is updated

    sudo apt update && sudo apt upgrade -y

Install weston

    sudo apt install weston

Install Waydroid

    curl -s https://repo.waydro.id | sudo bash
    sudo apt install waydroid

Run weston

    weston --backend=wayland-backend.so

Inside weston desktop shell open a terminal and run

    waydroid session start

When it shows 'Android with user 0 is ready' then open another terminal inside the weston desktop and run

    waydroid show-full-ui

## Tips:

- If Waydroid shows some weird dbus errors make sure you shutdown the WSL2 VM (wsl.exe --shutdown) and try again.

- By default Waydroid only launches correctly if the WSL2 networking mode is not MIRRORED. This is because dnsmasq tries to reserve some ports that are already used in Windows. If you want to keep using MIRRORED networking you will need to add this to your .wslconfig (or using WSL Settings)

  ```
  [experimental]
  ignoredPorts=53,67,68
  ```

- Every time the WSL2 kernel changes significantly (from ASHMEM support in 5.15.x to none in 6.6.x) you need to reconfigure Waydroid or it wont boot.

        sudo waydroid upgrade --offline
