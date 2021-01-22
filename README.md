

# Providing and configuring bootloaders for PXE clients
https://docs.fedoraproject.org/en-US/fedora/rawhide/install-guide/advanced/Network_based_Installations/

## Get the syslinux bootloader for BIOS clients.
```bash
# Create tftpboot, bios and uefi directories
mkdir -p /tftpboot/{uefi,pxelinux.cfg}

##### Download syslinux tar gz and extract the below files
https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/

# Install the syslinux package.
dnf install syslinux

# Copy bootloader files to tftpboot directory
cp /usr/share/syslinux/{lpxelinux.0,pxelinux.0,menu.c32,vesamenu.c32,ldlinux.c32,libcom32.c32,libutil.c32} /tftpboot/
```

## Get the bootloader files for UEFI systems
```bash
#
https://github.com/rhboot/shim/releases

# Install the shim-x64 and grub2-efi-x64 packages.
dnf install --installroot=/tmp/fedora \
  --releasever 32 \
  shim-x64 \
  grub2-efi-x64 \

# Copy bootloader files to tftpboot uefi directory
cp /tmp/fedora/boot/efi/EFI/fedora/{shimx64.efi,grubx64.efi} /tftpboot/uefi/
```

## Get the kernel and initrd

```bash
# Create a directory for the files.
mkdir -p /tftpboot/f32

https://mirror.netsite.dk/fedora/linux/releases/32/Server/x86_64/os/images/pxeboot/

# Download the kernel and initrd
## Fedora
https://download.fedoraproject.org/pub/fedora/linux/releases/32/Server/x86_64/os/images/pxeboot/
## CentOS
http://mirror.centos.org/centos/8-stream/BaseOS/x86_64/os/images/pxeboot/


```

## Configuring client bootloaders
```bash
# Create a boot menu for BIOS clients
echo > /tftpboot/pxelinux.cfg/default <<EOF
default vesamenu.c32
prompt 0
timeout 50

label server
  menu label ^Install Fedora 32 ( Minimal Image )
  kernel f32/vmlinuz
  append initrd=f32/initrd.img inst.stage2=https://download.fedoraproject.org/pub/fedora/linux/releases/32/Server/x86_64/os/ ip=dhcp ks=ftp://10.133.250.202/tftpboot/f32/kickstart.cfg

label linux
  menu label ^Install Fedora 32 ( Graphical )
  kernel f32/vmlinuz
  append initrd=f32/initrd.img inst.stage2=https://download.fedoraproject.org/pub/fedora/linux/releases/32/Server/x86_64/os/ ip=dhcp

label local
  menu label Boot from ^local drive
  localboot 0xffff
EOF

# Create a boot menu for UEFI clients

echo > /tftpboot/uefi/grub.cfg <<EOF
function load_video {
	insmod efi_gop
	insmod efi_uga
	insmod video_bochs
	insmod video_cirrus
	insmod all_video
}

load_video
set gfxpayload=keep
insmod gzio

menuentry 'Install Fedora 32 ( Minimal Image )'  --class fedora --class gnu-linux --class gnu --class os {
	kernel tftpboot/f32/vmlinuz
	append initrd=tftpboot/f32/initrd.img inst.repo=https://download.fedoraproject.org/pub/fedora/linux/releases/32/Server/x86_64/os/ ip=dhcp ks=ftp://10.133.250.202/tftpboot/f32/kickstart.cfg
}

menuentry 'Install Fedora 64-bit'  --class fedora --class gnu-linux --class gnu --class os {
	linuxefi tftpboot/f32/vmlinuz inst.repo=https://download.fedoraproject.org/pub/fedora/linux/releases/32/Server/x86_64/os/ ip=dhcp
	initrdefi tftpboot/f32/initrd.img
}

menuentry 'Exit this grub' {
	exit
}
EOF

```

## Copy to FTP / TFTP Server
Copy all the content from `tftpboot` on the FTP / TFTP Server into `tftpboot` directory.

# Fedora Core OS
Documentation for installer:
- https://coreos.github.io/coreos-installer/getting-started/

## Get kernel and images
```bash
podman run --privileged --pull=always --rm -v ${PWD}:/data -w /data  quay.io/coreos/coreos-installer:release download -f pxe
```

## Fedora CoreOS Config (FCC) & Ignition File
- https://docs.fedoraproject.org/en-US/fedora-coreos/fcct-config/
- https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/
- https://www.matthiaspreu.com/posts/fedora-coreos-first-steps/

```bash
# generate password_hash
mkpasswd --method=SHA-512 --rounds=4096 password

# Generate Ignitation file out of FCC
podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < fcos.fcc > fcos.ign
```

## Login
```bash
# Set keymap
sudo localectl set-keymap de

```