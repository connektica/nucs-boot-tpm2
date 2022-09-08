
# Systemd-CryptSetup operation combined with initramfs-tools

## Installation:
### TLDR:
If you just 'want it to work'  then run `sudo ./install.sh`.  and everything *should* work.  It will install Docker on your host system and then do all the work inside docker, so there is minimal impact.  the CWD will get some extra packages added to it, plus some extra directories with source files inside, but you can ignore all of that - once the script has completed successfully this entire directory can be removed.

### I want to understand!
0. Read the scripts for full details of what's happening.  They've been documented by function names, and should be reasonably easy to understand both what's happening and why it is happening.
	start with install.sh 'tldr_just_Work' and read the rest of the functions from there.
1. cryptroot
	replaces /usr/share/initramfs/local-top/cryptroot
2. cryptsetup_functions
	replaces /usr/lib/cryptsetup/functions.sh
3. systemd_cryptsetup_hook
	adds to /etc/initramfs-tools/hooks

## Operation:
These files build on the 'cryptroot' module in initramfs-tools/cryptsetup to allow the ```bash /etc/crypttab``` file to support the 'tpm-device=xxx' option and pass it through to the systemd-crypsetup application to allow automatically decrypting a device using LUKS and a TPM.


## Compilation:

To compile Systemd with TPM2 support, the script ```build_systemd_with_tpm2_support.sh``` can be used to either build inside a fresh ubuntu:22.04 image, or on the host.

This will create the deb files for system in the current working directory.


To build inside a fresh docker image:
./build_systemd_with_tpm2_support.sh

Or, to build on this host directly:
./build_systemd_with_tpm2_support.sh on_this_host



# Licensing #
I've been unable to establish who publishes the files I've patched and how they are licensed, so I've only included the patch here so I do not infringe anyone's copyright or break licensing.



# Full Process #
This is a rough and ready summary of the process to get a system running with an encrypted root.  It isn't as detailed as I'd like and there are more ways to get this to work (Work in Progress!).
A lot of this has been developed from the excellent articles in Arch Linux, so if there are steps missing you may need to read around the issues, or ask in the 'Issues' and someone (possibly me) will answer when they can.

Potentially useful pages for more context:
https://wiki.archlinux.org/title/Trusted_Platform_Module#Data-at-rest_encryption_with_LUKS
https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system
https://wiki.archlinux.org/title/Data-at-rest_encryption
(big thanks to the authors of these articles - they helped me get most of the way here!)


1. create partition format with LUKS (using an Ubuntu Desktop live environment some of this may need to be done from the command line then start the installer)
   1.1. assign space for unencrypted EFI system partition (stores grub2, Linux Kernel and initrd image or other system software)
   1.2. assign space for the LVM PV and any other un-encrypted partitions you may want
   1.3. assign space for the un-encrypted /boot partition (which may be removed if using a unified kernel image after the system has been configured) - done at the end to allow extending the PV partition in to it with pvresize.
2. Use lvm
   2.1. pvcreate
   2.2. vgcreate
   2.3. lvcreate
3. Install Ubuntu in to correct LV and unencrypted EFI system and /boot partitions!
4. Reboot in to the new Ubuntu environment:
    - the system halts in the initrd shell as it does not know how to unlock the LUKS (cryptrd not created) and find the LV  used as root.
    - the user has to manually unlock the LUKS partition with cryptsetup, then exit the shell and the system continues to boot.
6. Install git, get this repo, create the crypttab, run install.sh
7. Store a key in the TPM for LUKS
   `systemd-cryptenroll`
8. Reboot.


NB:  This doesn't protect against a modified initrd as yet.  That's another stage of configuring secure boot and creating an (optionally signed) Unified Kernel Image (or enabling key verification with GRUB2 or something along those lines!)  But this is a good step on the way!