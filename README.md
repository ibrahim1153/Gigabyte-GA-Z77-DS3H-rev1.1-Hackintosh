# Gigabyte-GA-Z77-DS3H-rev1.1-Hackintosh

Hackintosh for [Gigabyte GA-Z77-DS3H rev1.1](http://www.gigabyte.com/products/product-page.aspx?pid=4326) motherboard using OS X 10.10 Yosemite.

[Intel Z77 chipset](https://ark.intel.com/products/64024), [LGA 1155 socket](https://en.wikipedia.org/wiki/LGA_1155).  
Supports 3rd gen. ([22 nm - Ivy Bridge](http://en.wikipedia.org/wiki/Ivy_Bridge_(microarchitecture))) and 2nd gen. ([32 nm - Sandy Bridge](http://en.wikipedia.org/wiki/Sandy_Bridge)) Intel Core CPUs.

Onboard devices:
- Qualcomm Atheros AR8161 Gigabit Ethernet controller (DS3H rev1.0 has AR8151)
- Realtek ALC887 audio chipset

This is a minimal guide that fits my hardware configuration:
- [Intel Core i7-3770 with HD Graphics 4000](https://ark.intel.com/products/65719)
- [Nvidia GT 640](https://www.asus.com/Graphics-Cards/GT640DCSL2GD3/)
- [802.11ac WiFi card Broadcom BCM94360CD](#10)
- [SSD Crucial MX500](http://www.crucial.com/ProductDisplay?catalogId=10151&productId=2428501)

## BIOS Settings

Latest stable BIOS: version [F9 (2012/09/27 update)](https://www.gigabyte.com/Motherboard/GA-Z77-DS3H-rev-11#support-dl-bios)
- Save & Exit > Load Optimized Defaults
- Peripherals > SATA Mode Selection - AHCI
- BIOS Features > Intel Virtualization Technology - Disabled (or add kernel flag [`dart=0`](#1) to `CLOVER/config.plist`)
- BIOS Features > VT-d - Disabled (or add kernel flag `dart=0`)

Note: [Intel Virtualization Technology (VT-x)](http://en.wikipedia.org/wiki/X86_virtualization#Intel_virtualization_.28VT-x.29)
is supported by almost every Intel [Sandy Bridge](http://en.wikipedia.org/wiki/Sandy_Bridge_%28microarchitecture%29)
and [Ivy Bridge](http://en.wikipedia.org/wiki/Ivy_Bridge_%28microarchitecture%29) processors.
This is not the case for [I/O MMU virtualization (VT-d)](http://en.wikipedia.org/wiki/X86_virtualization#I.2FO_MMU_virtualization_.28AMD-Vi_and_VT-d.29).

Sources:
- [How to set up the UEFI of your Hackintosh's Gigabyte motherboard](http://www.macbreaker.com/2012/08/set-up-hackintosh-gigabyte-uefi.html)
- [BIOS/UEFI Screenshots - Gigabyte Z77X-UP5-TH](http://www.tonymacx86.com/bios-uefi/130888-bios-uefi-screenshots-gigabyte-z77x-up5-th.html)

## DSDT

See DSDT/PJALM/GA-Z77-DS3H.txt

## Clover

Sources:
- [Ramblings of a Hackintosher - A (Sorta) Brief Vanilla Install Guide](https://www.reddit.com/r/hackintosh/comments/68p1e2/ramblings_of_a_hackintosher_a_sorta_brief_vanilla/)
- [Clover Wiki](https://clover-wiki.zetam.org)
- [2017/06/24 - My Main Hackintosh Desktop Sierra 10.12.5 Z77-DS3H](https://voiletdragon.wordpress.com/2017/06/24/guide-my-main-hackintosh-desktop-sierra-10-12-5-z77-ds3h/)
- [GitHub VoiletDragon/Z77-DS3H-Clover-Hotpatch-Patches](https://github.com/VoiletDragon/Z77-DS3H-Clover-Hotpatch-Patches)

https://www.reddit.com/r/hackintosh/comments/7cuccm/gaz77xd3h_high_sierra_success/

### Create a bootable installer

Becarefull, this will erase the disk

```
diskutil list
diskutil partitionDisk /dev/disk# GPT JHFS+ "USB" 100%
```

```
open http://appstore.com/mac/macoshighsierra
[...]
sudo "/Applications/Install macOS High Sierra.app/Contents/Resources/createinstallmedia" --volume /Volumes/USB --applicationpath "/Applications/Install macOS High Sierra.app" --nointeraction
```

- [How to create a bootable installer for macOS](https://support.apple.com/en-us/HT201372)

### Clover Installer

```
curl -O https://netix.dl.sourceforge.net/project/cloverefiboot/Installer/Clover_v2.4k_r4617.zip
unzip Clover_v2.4k_r4617.zip
open Clover_v2.4k_r4617.pkg
```

- Change Install Location... > Install macOS High Sierra
- Customize >
  - [X] Install for UEFI booting only
  - [X] Install Clover in the ESP
  - Drivers64UEFI

    Beside default drivers:
    - [X] ApfsDriverLoader-64 (allows Clover to load APFS volumes thanks to apfs.efi)
    - [X] AptioMemoryFix-64 (fixes memory problems with AMI UEFI BIOS Aptio)
    - [X] VBoxHfs-64.efi (support for HFS+, not needed if using APFS)

```
diskutil eject EFI
```

### apfs.efi

```
# cp /usr/standalone/i386/apfs.efi /Volumes/EFI/EFI/CLOVER/drivers64UEFI
open "/Applications/Install macOS High Sierra.app/Contents/SharedSupport/BaseSystem.dmg"
cp "/Volumes/OS X Base System/usr/standalone/i386/apfs.efi" /Volumes/EFI/EFI/CLOVER/drivers64UEFI
diskutil eject "OS X Base System"
perl -i -pe 's|\x00\x74\x07\xb8\xff\xff|\x00\x90\x90\xb8\xff\xff|sg' /Volumes/EFI/EFI/CLOVER/drivers64UEFI/apfs.efi
```

- [Hackintosher 2018/03/19 - How-to update and patch apfs.efi on a Hackintosh](https://hackintosher.com/forums/thread/how-to-update-and-patch-apfs-efi-on-a-hackintosh.126/)
- [GitHub corpnewt/APFS-Non-Verbose](https://github.com/corpnewt/APFS-Non-Verbose)
- [GitHub JennyDavid/Apfs.efi-for-macOS-High-Sierra](https://github.com/JennyDavid/Apfs.efi-for-macOS-High-Sierra)

### FakeSMC.kext

```
curl -OL https://bitbucket.org/RehabMan/os-x-fakesmc-kozlek/downloads/RehabMan-FakeSMC-2018-0403.zip
unzip RehabMan-FakeSMC-2018-0403.zip -d FakeSMC
cp -R FakeSMC/*.kext /Volumes/EFI/EFI/CLOVER/kexts/Other
cp -R FakeSMC/HWMonitor.app /Applications
```

- [GitHub RehabMan/OS-X-FakeSMC-kozlek](https://github.com/RehabMan/OS-X-FakeSMC-kozlek)

### AppleALC.kext and Lilu.kext

```
curl -OL https://github.com/acidanthera/AppleALC/releases/download/1.3.0/1.3.0.RELEASE.zip
unzip 1.3.0.RELEASE.zip
cp -R AppleALC.kext /Volumes/EFI/EFI/CLOVER/kexts/Other
```

```
curl -OL https://github.com/acidanthera/Lilu/releases/download/1.2.5/1.2.5.RELEASE.zip
unzip 1.2.5.RELEASE.zip
cp -R Lilu.kext /Volumes/EFI/EFI/CLOVER/kexts/Other
```

- [GitHub acidanthera/AppleALC](https://github.com/acidanthera/AppleALC)

### AtherosE2200Ethernet.kext

[AtherosE2200Ethernet](https://github.com/Mieze/AtherosE2200Ethernet) is the most up to date and stable driver for Qualcomm Atheros AR8161 Ethernet controller

```
git clone https://github.com/Mieze/AtherosE2200Ethernet.git
xcodebuild -project AtherosE2200Ethernet/AtherosE2200Ethernet.xcodeproj
cp -R AtherosE2200Ethernet/build/Release/AtherosE2200Ethernet.kext /Volumes/EFI/EFI/CLOVER/kexts/Other
```

## SSDT

For proper CPU power management, you should [generate a SSDT](#2) (otherwise [Intel Turbo Boost](https://en.wikipedia.org/wiki/Intel_Turbo_Boost) won't work).

Clover config.plist parameter `PluginType` (supposed to enable CPU power management) did not work.

```
curl -O https://raw.githubusercontent.com/Piker-Alpha/ssdtPRGen.sh/Beta/ssdtPRGen.sh
chmod +x ssdtPRGen.sh
./ssdtPRGen.sh -m iMac13,2 -target 1 # 1 = Ivy Bridge
[...]
cp ~/Library/ssdtPRGen/ssdt.aml /Volumes/EFI/EFI/CLOVER/ACPI/patched/
```

Sources:
- [hackintosher.com 2017/11 - Generating a Coffee Lake SSDT on a Hackintosh](https://hackintosher.com/guides/generating-coffee-lake-ssdt-hackintosh/)
- [ssdtPRGen.sh - Script to generate a SSDT for Power Management](https://github.com/Piker-Alpha/ssdtPRGen.sh)

### config.plist

```
curl https://raw.githubusercontent.com/tkrotoff/Gigabyte-GA-Z77-DS3H-rev1.1-Hackintosh/master/config.template.plist

uuidgen

curl -OL https://github.com/acidanthera/macserial/releases/download/2.0.2/macserial-2.0.2-mac.zip
unzip macserial-2.0.2-mac.zip
./macserial --generate | head -n 1 | awk '{print substr($0,0,12)}'
| perl -pe 's|<string>000000000000</string>|$_|sg' < config.template.plist > config.plist

plutil config.plist
cp config.plist /Volumes/EFI/EFI/CLOVER
```

Sources:
- [reddit 2015/06/30 - iMessage with Clover](https://www.reddit.com/r/hackintosh/comments/3bjxhl/dual_boot_issue_noob_question/csn8nfd)
- [tonymac 2018/07/08 - An iDiot's Guide To iMessage](https://www.tonymacx86.com/threads/an-idiots-guide-to-imessage.196827/)
- [tonymac 2017/05/23 - How to Fix iMessage](https://www.tonymacx86.com/threads/how-to-fix-imessage.110471/)
- [tonymac 2015/10/20 - Clover DSDT Fixes](https://www.tonymacx86.com/threads/clover-dsdt-fixes.176195/)

## Performance

Using [Geekbench 4](http://www.primatelabs.com/geekbench/), you should get a score (Intel Core i7-3770 @ 3.40 GHz) > 4000 (single-core) > 14000 (multi-core), see issue #2.
https://browser.geekbench.com/v4/cpu/9088039
https://browser.geekbench.com/v4/cpu/9089562
https://browser.geekbench.com/v4/cpu/9089632

## Tips

### Mount EFI partition

```
diskutil list
sudo diskutil mount /dev/disk#s1
```

### Logs

Clover boot logs:
```
bdmesg
```

```
git clone https://github.com/corpnewt/EssentialsList.git
chmod +x EssentialsList/EssentialsList.command
./EssentialsList/EssentialsList.command
```

### Boot flags

If the system does not boot (crash), flags `-v` (verbose), `-x` (safe mode) and `-s` (single user mode - gives you a Unix shell) can help.

Example with `/Volumes/EFI/EFI/CLOVER/config.plist`:
```XML
<key>Arguments</key>
<string>dart=0 -v</string>
```

Sources:
- [Clover boot configuration](https://clover-wiki.zetam.org/Configuration/Boot).
- [Hackintosh Boot Flags](http://www.fitzweekly.com/2016/04/hackintosh-boot-flags.html)

### Prevent macOS from mounting a volume

```
sudo vifs
```

Example:
```
# Do not mount NTFS disk 'Windows 7 Boot'
LABEL=Windows\0407\040Boot none ntfs ro,noauto

# Do not mount NTFS disk 'HD502HJ'
LABEL=HD502HJ none ntfs ro,noauto

# Mount NTFS disk 'HD204UI' in read-write mode (experimental: at your own risk)
# Option 'nobrowse' is mandatory. You will have to manually open /Volumes/HD204UI
# using the Finder or Disk Utility
LABEL=HD204UI none ntfs rw,auto,nobrowse

# Do not mount ExFAT disk 'WD20EZRX'
LABEL=WD20EZRX none exfat rw,noauto
```

Sources:
- [Write in NTFS using Mavericks](http://apple.stackexchange.com/a/112990)
- [Prevent a partition from mounting in OS X](http://www.cnet.com/how-to/prevent-a-partition-from-mounting-in-os-x/)

### Create a bootable Windows 10 USB key

```Shell
hdiutil convert -format UDRW -o Win10.img /PATH/Win10_XXX.iso
diskutil list
diskutil unmountDisk /dev/diskXXX
sudo dd if=Win10.img.dmg of=/dev/diskXXX bs=1m
sudo dd if=Downloads/Win10_1607_SingleLang_French_x64.iso of=/dev/rdisk5 bs=1m
diskutil eject /dev/diskXXX
```

Sources:
- [Creating A Bootable USB Of Windows 8.1 On OS X?](http://apple.stackexchange.com/q/103874/150369)
- [How to create a bootable Windows 10 USB in OS X using Terminal](https://www.tonymacx86.com/threads/how-to-create-a-bootable-windows-10-usb-in-os-x-using-terminal.172458/page-3#post-1317152)
- [Creating a Windows 7 USB installation disk on a Mac](http://superuser.com/q/133152/505295)

## Other tools and links

- [Clover EFI bootloader project page](http://sourceforge.net/projects/cloverefiboot/)
- [Clover Configurator](https://mackie100projects.altervista.org/clover-configurator/): graphical editor for Clover config.plist

## License

Do whatever you like, this is public domain.
