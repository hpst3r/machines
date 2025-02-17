# Fetch

```
[liam@m715q-0 ~]$ fastfetch
         'c:.                               liam@m715q-0
        lkkkx, ..       ..   ,cc,           ------------
        okkkk:ckkx'  .lxkkx.okkkkd          OS: AlmaLinux 9.5 (Teal Serval) x86_64
        .:llcokkx'  :kkkxkko:xkkd,          Host: 10VHS1P100 (ThinkCentre M715q)
      .xkkkkdood:  ;kx,  .lkxlll;           Kernel: Linux 5.14.0-503.11.1.el9_5.x86_64
       xkkx.       xk'     xkkkkk:          Uptime: 5 hours, 10 mins
       'xkx.       xd      .....,.          Packages: 774 (rpm)
      .. :xkl'     :c      ..''..           Shell: bash 5.1.8
    .dkx'  .:ldl:'. '  ':lollldkkxo;        Display (PGS030F): 1280x1024 @ 60 Hz in 19" [External]
  .''lkkko'                     ckkkx.      Cursor: Adwaita
'xkkkd:kkd.       ..  ;'        :kkxo.      Terminal: /dev/pts/0
,xkkkd;kk'      ,d;    ld.   ':dkd::cc,     CPU: AMD Ryzen 5 PRO 2400GE w/ Radeon Vega Graphics (8) @ 3.20 GHz
 .,,.;xkko'.';lxo.      dx,  :kkk'xkkkkc    GPU: AMD Radeon Vega 11 Graphics [Integrated]
     'dkkkkkxo:.        ;kx  .kkk:;xkkd.    Memory: 707.30 MiB / 15.06 GiB (5%)
       .....   .;dk:.   lkk.  :;,           Swap: 0 B / 16.00 GiB (0%)
             :kkkkkkkdoxkkx                 Disk (/): 2.66 GiB / 82.13 GiB (3%) - ext4
              ,c,,;;;:xkkd.                 Disk (/home): 76.00 KiB / 4.03 GiB (0%) - ext4
                ;kkkkl...                   Disk (/var): 1.05 GiB / 9.75 GiB (11%) - ext4
                ;kkkkl                      Local IP (enp1s0f0): 192.168.77.147/24
                 ,od;                       Locale: C.UTF-8
```

# Introduction

This machine is a Lenovo M715q Gen 2. It's main claim to fame is that it is a Zen 1 Tiny Mini Micro (TMM) node, which were not overly popular.

I purchased it off eBay for about $50 in late 2024 with a single 8gb SODIMM. I upgraded it with a second 8gb SODIMM, then swapped in an AX200 (surprisingly, it came with an Intel 9260NGW) and installed a 120gb NVME SSD I had lying around.

This system boasts a real monster of a quad core Zen 1 part: the mighty 35w Ryzen 5 2400GE. Back in its day this thing had decent integrated graphics.. and I think the praise stops there. But, well, it was a first-gen product, and we know how well Zen is doing today..

Unlike the i5-8600T ThinkCentre M920qs that I have around, this guy is not particularly fast, particularly expandable or particularly interesting:
- It lacks the internal PCI-E 3.0x8 slot that the M920q and M720q both feature - no cheeky i350 or X540 upfits here
- does not have vPro support like the M920q (it has DASH, but, well..)
- has a noticeably cheaper chassis (reused from AMD A-series APU ThinkCentre Tiny models)
- features an annoyingly loud cooling fan
- has a Realtek Ethernet chipset, not an Intel one (RTL8111)

After installing and provisioning this machine with one of my playbooks, I noticed some errors:

Some Vital Product Data (VPD - basic PCIe bus config info) access errors:

```txt
Feb 16 17:58:03 m715q-0 kernel: serial 0000:01:00.1: VPD access failed.  This is likely a firmware bug on this device.  Contact the card vendor for a firmware update
Feb 16 17:58:04 m715q-0 kernel: serial 0000:01:00.2: VPD access failed.  This is likely a firmware bug on this device.  Contact the card vendor for a firmware update
Feb 16 17:58:04 m715q-0 kernel: pci 0000:01:00.3: VPD access failed.  This is likely a firmware bug on this device.  Contact the card vendor for a firmware update
Feb 16 17:58:04 m715q-0 kernel: ehci-pci 0000:01:00.4: VPD access failed.  This is likely a firmware bug on this device.  Contact the card vendor for a firmware update
```

These PCIe devices are, respectively:

```
01:00.1 Serial controller: Realtek Semiconductor Co., Ltd. RTL8111xP UART #1 (rev 0e)
01:00.2 Serial controller: Realtek Semiconductor Co., Ltd. RTL8111xP UART #2 (rev 0e)
01:00.3 IPMI Interface: Realtek Semiconductor Co., Ltd. RTL8111xP IPMI interface (rev 0e)
01:00.4 USB controller: Realtek Semiconductor Co., Ltd. RTL811x EHCI host controller (rev 0e)
```

Good old Realtek Linux drivers. Never gets old (/s).

TSC seems to be a problem:

```txt
Feb 16 17:44:24 m715q-0 kernel: clocksource: timekeeping watchdog on CPU4: Marking clocksource 'tsc' as unstable because the skew is too large:
Feb 16 17:44:24 m715q-0 kernel: clocksource:                       'hpet' wd_nsec: 495115789 wd_now: 22a1e0a wd_last: 1bdf205 mask: ffffffff
Feb 16 17:44:24 m715q-0 kernel: clocksource:                       'tsc' cs_nsec: 495992089 cs_now: 103ae0e660 cs_last: fdc73efe0 mask: ffffffffffffffff
Feb 16 17:44:24 m715q-0 kernel: clocksource:                       'tsc' is current clocksource.
Feb 16 17:44:24 m715q-0 kernel: tsc: Marking TSC unstable due to clocksource watchdog
Feb 16 17:44:24 m715q-0 kernel: TSC found unstable after boot, most likely due to broken BIOS. Use 'tsc=unstable'.
Feb 16 17:44:24 m715q-0 kernel: sched_clock: Marking unstable (2515210507, -8992143)<-(2514285716, -8067321)
```

According to a brief search, this is pretty common with Ryzen CPUs running Linux. I don't recall noticing this with my R5 1600AF machine, but I probably just ignored it.

Some links to other occurrences:
- [linuxquestions.org - 3400G, Turion 64](https://www.linuxquestions.org/questions/slackware-14/tsc-found-unstable-after-boot-4175711883/)
- [reddit.com - 5750G, 4700U, 5700G, 5800H](https://www.reddit.com/r/linuxquestions/comments/ts1hgw/zen_3_tsc_unstable_which_part_to_blame/?rdt=55114)

Both posts suggest doing the same thing - setting clocksource=tsc tsc=reliable in boot params - to get rid of this. As I don't have any PCIe devices that really matter, and performance seems fine to me, I'm going to leave it be for now.

I've loaded AlmaLinux 9 on this machine with the Cockpit web UI, plus the Cockpit-Machines and Cockpit-Podman plugins for easy virtual machine and container management. I make a few tweaks to the system and install software with a fairly standard Ansible playbook that is used to configure all my AlmaLinux and Fedora Linux hypervisors.

# Usage

Before this most recent wipe, I had this box running some supporting Windows infrastructure for a lab environment. While 16gb of RAM is a bit tight, and that annoying little fan did make itself known at times, it did the trick.

I wiped it, reinstalled it with EXT4 on LVM (vs its former configuration with XFS after a wholly avoidable power-loss incident) and now it's in the state you see here.

The 2400GE is adequate, even if it leaves a bit to be desired. It barely squeaks past my old E5-2660 v3 servers in single-core performance and is brutalized by the boxes with five times the core count in multi-core workloads, which.. well, is to be expected.

Windows Server has some issues on this machine. Running Geekbench 6 (a benchmark that really only cares about memory bandwidth - not the greatest, but an easy sanity check) on Windows Server 2025 and AlmaLinux 9 show a vast difference in performance:

### Windows Server 2025

Single core: 658
Multi core: 2072
[Link](https://browser.geekbench.com/v6/cpu/10051652)

### AlmaLinux 9.5

Single core: 1093
Multi core: 3092
[Link](https://browser.geekbench.com/v6/cpu/10051921)

### Reference performance

A Lenovo ThinkCentre M920q with an i5-8600T (6 cores, 6 threads) and dual-channel DDR4 2666 (2x16gb) running Fedora 40 [scores about 1300 single/4300 multi.](https://browser.geekbench.com/v6/cpu/7863658) This machine ran me around $100.

A SuperMicro X10DRL-i with a pair of E5-2660 v3s (2x 10 cores, 20 threads) and quad-channel memory per CPU (8x16gb) running Fedora 40 [scores about 1075 single/8000 multi.](https://browser.geekbench.com/v6/cpu/7480724) This machine ran me around $150.

The difference between Fedora and AlmaLinux is typically minor, but I will note that Fedora 40 uses a much newer kernel than Alma 9 - 6.6 vs 5.14, respectively.

### My thoughts

Overall, it's a fine little box, though it makes very little sense to go for one of these over a M710q/M910q (which will be quieter, cheaper, have a nicer case, and are typically slightly less expensive).

Maybe some day I'll come back and run some more thorough benchmarks here. I'd like to get a standard suite going.

# System info
## lspci

```txt
[liam@m715q-0 ~]$ lspci
00:00.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Root Complex
00:00.2 IOMMU: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 IOMMU
00:01.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
00:01.2 PCI bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 PCIe GPP Bridge [6:0]
00:01.3 PCI bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 PCIe GPP Bridge [6:0]
00:01.4 PCI bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 PCIe GPP Bridge [6:0]
00:08.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
00:08.1 PCI bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Internal PCIe GPP Bridge 0 to Bus A
00:08.2 PCI bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Internal PCIe GPP Bridge 0 to Bus B
00:14.0 SMBus: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller (rev 61)
00:14.3 ISA bridge: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge (rev 51)
00:18.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 0
00:18.1 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 1
00:18.2 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 2
00:18.3 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 3
00:18.4 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 4
00:18.5 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 5
00:18.6 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 6
00:18.7 Host bridge: Advanced Micro Devices, Inc. [AMD] Raven/Raven2 Device 24: Function 7
01:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8211/8411 PCI Express Gigabit Ethernet Controller (rev 0e)
01:00.1 Serial controller: Realtek Semiconductor Co., Ltd. RTL8111xP UART #1 (rev 0e)
01:00.2 Serial controller: Realtek Semiconductor Co., Ltd. RTL8111xP UART #2 (rev 0e)
01:00.3 IPMI Interface: Realtek Semiconductor Co., Ltd. RTL8111xP IPMI interface (rev 0e)
01:00.4 USB controller: Realtek Semiconductor Co., Ltd. RTL811x EHCI host controller (rev 0e)
02:00.0 Network controller: Intel Corporation Wi-Fi 6 AX200 (rev 1a)
03:00.0 Non-Volatile memory controller: SK hynix Gold P31/BC711/PC711 NVMe Solid State Drive
04:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Raven Ridge [Radeon Vega Series / Radeon Vega Mobile Series] (rev d6)
04:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Raven/Raven2/Fenghuang HDMI/DP Audio Controller
04:00.2 Encryption controller: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 10h-1fh) Platform Security Processor
04:00.3 USB controller: Advanced Micro Devices, Inc. [AMD] Raven USB 3.1
04:00.4 USB controller: Advanced Micro Devices, Inc. [AMD] Raven USB 3.1
04:00.5 Multimedia controller: Advanced Micro Devices, Inc. [AMD] ACP/ACP3X/ACP6x Audio Coprocessor
04:00.6 Audio device: Advanced Micro Devices, Inc. [AMD] Family 17h/19h HD Audio Controller
05:00.0 SATA controller: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] (rev 61)
```

## lscpu

```txt
[liam@m715q-0 ~]$ lscpu
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          43 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   8
  On-line CPU(s) list:    0-7
Vendor ID:                AuthenticAMD
  Model name:             AMD Ryzen 5 PRO 2400GE w/ Radeon Vega Graphics
    CPU family:           23
    Model:                17
    Thread(s) per core:   2
    Core(s) per socket:   4
    Socket(s):            1
    Stepping:             0
    Frequency boost:      enabled
    CPU(s) scaling MHz:   84%
    CPU max MHz:          3200.0000
    CPU min MHz:          1600.0000
    BogoMIPS:             6387.89
    Flags:                fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep
                          _good nopl nonstop_tsc cpuid extd_apicid aperfmperf rapl pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_leg
                          acy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb hw_pstate ssbd ibp
                          b vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt sha_ni xsaveopt xsavec xgetbv1 clzero irperf xsaveerptr arat npt lbrv svm_lock nrip_save tsc_sca
                          le vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif overflow_recov succor smca sev sev_es
Virtualization features:
  Virtualization:         AMD-V
Caches (sum of all):
  L1d:                    128 KiB (4 instances)
  L1i:                    256 KiB (4 instances)
  L2:                     2 MiB (4 instances)
  L3:                     4 MiB (1 instance)
NUMA:
  NUMA node(s):           1
  NUMA node0 CPU(s):      0-7
Vulnerabilities:
  Gather data sampling:   Not affected
  Itlb multihit:          Not affected
  L1tf:                   Not affected
  Mds:                    Not affected
  Meltdown:               Not affected
  Mmio stale data:        Not affected
  Reg file data sampling: Not affected
  Retbleed:               Mitigation; untrained return thunk; SMT vulnerable
  Spec rstack overflow:   Mitigation; Safe RET
  Spec store bypass:      Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:             Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:             Mitigation; Retpolines; IBPB conditional; STIBP disabled; RSB filling; PBRSB-eIBRS Not affected; BHI Not affected
  Srbds:                  Not affected
  Tsx async abort:        Not affected
```

## /proc/cpuinfo

```txt
[liam@m715q-0 ~]$ cat /proc/cpuinfo
processor       : 0
vendor_id       : AuthenticAMD
cpu family      : 23
model           : 17
model name      : AMD Ryzen 5 PRO 2400GE w/ Radeon Vega Graphics
stepping        : 0
microcode       : 0x8101007
cpu MHz         : 3200.000
cache size      : 512 KB
physical id     : 0
siblings        : 8
core id         : 0
cpu cores       : 4
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 13
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf rapl pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb hw_pstate ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt sha_ni xsaveopt xsavec xgetbv1 clzero irperf xsaveerptr arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif overflow_recov succor smca sev sev_es
bugs            : sysret_ss_attrs null_seg spectre_v1 spectre_v2 spec_store_bypass retbleed smt_rsb srso div0
bogomips        : 6387.89
TLB size        : 2560 4K pages
clflush size    : 64
cache_alignment : 64
address sizes   : 43 bits physical, 48 bits virtual
power management: ts ttp tm hwpstate eff_freq_ro [13] [14]
```
## dmidecode

```txt
[liam@m715q-0 ~]$ sudo dmidecode
# dmidecode 3.6
Getting SMBIOS data from sysfs.
SMBIOS 3.1.1 present.
Table at 0x7CBE4000.

Handle 0x0000, DMI type 0, 26 bytes
BIOS Information
        Vendor: LENOVO
        Version: M1XKT39A
        Release Date: 04/08/2019
        Address: 0xF0000
        Runtime Size: 64 kB
        ROM Size: 16 MB
        Characteristics:
                PCI is supported
                BIOS is upgradeable
                BIOS shadowing is allowed
                Boot from CD is supported
                Selectable boot is supported
                BIOS ROM is socketed
                EDD is supported
                5.25"/1.2 MB floppy services are supported (int 13h)
                3.5"/720 kB floppy services are supported (int 13h)
                3.5"/2.88 MB floppy services are supported (int 13h)
                Print screen service is supported (int 5h)
                Serial services are supported (int 14h)
                Printer services are supported (int 17h)
                ACPI is supported
                USB legacy is supported
                BIOS boot specification is supported
                Targeted content distribution is supported
                UEFI is supported
        BIOS Revision: 0.39
        Firmware Revision: 0.12

Handle 0x0001, DMI type 1, 27 bytes
System Information
        Manufacturer: LENOVO
        Product Name: 10VHS1P100
        Version: ThinkCentre M715q
        Serial Number: MJ08LQZS
        UUID: 7cc43000-5a26-11e9-91e9-00db294e1b00
        Wake-up Type: Power Switch
        SKU Number: LENOVO_MT_10VH_BU_Think_FM_ThinkCentre M715q
        Family: ThinkCentre M715q

Handle 0x0002, DMI type 2, 15 bytes
Base Board Information
        Manufacturer: LENOVO
        Product Name: 3130
        Version: SDK0J40697 WIN 3305189113242
        Serial Number:
        Asset Tag:
        Features:
                Board is a hosting board
                Board is replaceable
        Location In Chassis: Default string
        Chassis Handle: 0x0003
        Type: Motherboard
        Contained Object Handles: 0

Handle 0x0003, DMI type 3, 22 bytes
Chassis Information
        Manufacturer: LENOVO
        Type: Mini PC
        Lock: Not Present
        Version: None
        Serial Number: MJ08LQZS
        Asset Tag:
        Boot-up State: Safe
        Power Supply State: Safe
        Thermal State: Safe
        Security Status: None
        OEM Information: 0x00000000
        Height: Unspecified
        Number Of Power Cords: 1
        Contained Elements: 0
        SKU Number: Default string

Handle 0x0004, DMI type 10, 6 bytes
On Board Device Information
        Type: Video
        Status: Enabled
        Description:    To Be Filled By O.E.M.

Handle 0x0005, DMI type 11, 5 bytes
OEM Strings
        String 1: LENOVO ThinkCentre Embedded Controller -[M1XCT12A-`.12]-
        String 2: LENOVO ThinkCentre BIOS Boot Block Revision 1.39
        String 3: Lenovo Service Engine Not Supported
        String 4: INVALID
        String 5: INVALID
        String 6: INVALID
        String 7: INVALID
        String 8: INVALID
        String 9: INVALID
        String 10: INVALID
        String 11: INVALID
        String 12: INVALID
        String 13: INVALID
        String 14: INVALID
        String 15: INVALID

Handle 0x0006, DMI type 12, 5 bytes
System Configuration Options
        Option 1: scre++

Handle 0x0007, DMI type 24, 5 bytes
Hardware Security
        Power-On Password Status: Disabled
        Keyboard Password Status: Enabled
        Administrator Password Status: Disabled
        Front Panel Reset Status: Not Implemented

Handle 0x0008, DMI type 32, 20 bytes
System Boot Information
        Status: No errors detected

Handle 0x0009, DMI type 34, 11 bytes
Management Device
        Description: LM78-1
        Type: LM78
        Address: 0x00000000
        Address Type: I/O Port

Handle 0x000A, DMI type 26, 22 bytes
Voltage Probe
        Description: CPU_CORE
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x000B, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x000C, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0009
        Component Handle: 0x000A
        Threshold Handle: 0x000B

Handle 0x000D, DMI type 28, 22 bytes
Temperature Probe
        Description: CPU TEMPERATURE
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x000E, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x000F, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0009
        Component Handle: 0x000D
        Threshold Handle: 0x000E

Handle 0x0010, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x000D
        Type: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: CPU FAN

Handle 0x0011, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0012, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0009
        Component Handle: 0x0010
        Threshold Handle: 0x0011

Handle 0x0013, DMI type 34, 11 bytes
Management Device
        Description: LM78-2
        Type: LM78
        Address: 0x00000000
        Address Type: I/O Port

Handle 0x0014, DMI type 26, 22 bytes
Voltage Probe
        Description: VCC_MEMORY
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0015, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0016, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x0014
        Threshold Handle: 0x0015

Handle 0x0017, DMI type 26, 22 bytes
Voltage Probe
        Description: 3D3V_S0
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0018, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0019, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x0017
        Threshold Handle: 0x0018

Handle 0x001A, DMI type 26, 22 bytes
Voltage Probe
        Description: VIN
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x001B, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x001C, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x001A
        Threshold Handle: 0x001B

Handle 0x001D, DMI type 26, 22 bytes
Voltage Probe
        Description: 5V_S0
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x001E, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x001F, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x001D
        Threshold Handle: 0x001E

Handle 0x0020, DMI type 26, 22 bytes
Voltage Probe
        Description: 1D05V
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0021, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0022, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x0020
        Threshold Handle: 0x0021

Handle 0x0023, DMI type 28, 22 bytes
Temperature Probe
        Description: SYSTEM TEMPERATURE
        Location: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Maximum Value: Unknown
        Minimum Value: Unknown
        Resolution: Unknown
        Tolerance: Unknown
        Accuracy: Unknown
        OEM-specific Information: 0x00000000
        Nominal Value: Unknown

Handle 0x0024, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0025, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x0023
        Threshold Handle: 0x0024

Handle 0x0026, DMI type 27, 15 bytes
Cooling Device
        Temperature Probe Handle: 0x0023
        Type: <OUT OF SPEC>
        Status: <OUT OF SPEC>
        Cooling Unit Group: 1
        OEM-specific Information: 0x00000000
        Nominal Speed: Unknown Or Non-rotating
        Description: Cooling Dev

Handle 0x0027, DMI type 36, 16 bytes
Management Device Threshold Data
        Lower Non-critical Threshold: 1
        Upper Non-critical Threshold: 2
        Lower Critical Threshold: 3
        Upper Critical Threshold: 4
        Lower Non-recoverable Threshold: 5
        Upper Non-recoverable Threshold: 6

Handle 0x0028, DMI type 35, 11 bytes
Management Device Component
        Description: PLDM Sensor
        Management Device Handle: 0x0013
        Component Handle: 0x0026
        Threshold Handle: 0x0027

Handle 0x0029, DMI type 18, 23 bytes
32-bit Memory Error Information
        Type: OK
        Granularity: Unknown
        Operation: Unknown
        Vendor Syndrome: Unknown
        Memory Array Address: Unknown
        Device Address: Unknown
        Resolution: Unknown

Handle 0x002A, DMI type 16, 23 bytes
Physical Memory Array
        Location: System Board Or Motherboard
        Use: System Memory
        Error Correction Type: None
        Maximum Capacity: 32 GB
        Error Information Handle: 0x0029
        Number Of Devices: 2

Handle 0x002B, DMI type 19, 31 bytes
Memory Array Mapped Address
        Starting Address: 0x00000000000
        Ending Address: 0x003FFFFFFFF
        Range Size: 16 GB
        Physical Array Handle: 0x002A
        Partition Width: 2

Handle 0x002C, DMI type 7, 19 bytes
Cache Information
        Socket Designation: L1 - Cache
        Configuration: Enabled, Not Socketed, Level 1
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 384 kB
        Maximum Size: 384 kB
        Supported SRAM Types:
                Pipeline Burst
        Installed SRAM Type: Pipeline Burst
        Speed: 1 ns
        Error Correction Type: Multi-bit ECC
        System Type: Unified
        Associativity: 8-way Set-associative

Handle 0x002D, DMI type 7, 19 bytes
Cache Information
        Socket Designation: L2 - Cache
        Configuration: Enabled, Not Socketed, Level 2
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 2 MB
        Maximum Size: 2 MB
        Supported SRAM Types:
                Pipeline Burst
        Installed SRAM Type: Pipeline Burst
        Speed: 1 ns
        Error Correction Type: Multi-bit ECC
        System Type: Unified
        Associativity: 8-way Set-associative

Handle 0x002E, DMI type 7, 19 bytes
Cache Information
        Socket Designation: L3 - Cache
        Configuration: Enabled, Not Socketed, Level 3
        Operational Mode: Write Back
        Location: Internal
        Installed Size: 4 MB
        Maximum Size: 4 MB
        Supported SRAM Types:
                Pipeline Burst
        Installed SRAM Type: Pipeline Burst
        Speed: 1 ns
        Error Correction Type: Multi-bit ECC
        System Type: Unified
        Associativity: 16-way Set-associative

Handle 0x002F, DMI type 4, 48 bytes
Processor Information
        Socket Designation: AM4
        Type: Central Processor
        Family: Zen
        Manufacturer: Advanced Micro Devices, Inc.
        ID: 10 0F 81 00 FF FB 8B 17
        Signature: Family 23, Model 17, Stepping 0
        Flags:
                FPU (Floating-point unit on-chip)
                VME (Virtual mode extension)
                DE (Debugging extension)
                PSE (Page size extension)
                TSC (Time stamp counter)
                MSR (Model specific registers)
                PAE (Physical address extension)
                MCE (Machine check exception)
                CX8 (CMPXCHG8 instruction supported)
                APIC (On-chip APIC hardware supported)
                SEP (Fast system call)
                MTRR (Memory type range registers)
                PGE (Page global enable)
                MCA (Machine check architecture)
                CMOV (Conditional move instruction supported)
                PAT (Page attribute table)
                PSE-36 (36-bit page size extension)
                CLFSH (CLFLUSH instruction supported)
                MMX (MMX technology supported)
                FXSR (FXSAVE and FXSTOR instructions supported)
                SSE (Streaming SIMD extensions)
                SSE2 (Streaming SIMD extensions 2)
                HTT (Multi-threading)
        Version: AMD Ryzen 5 PRO 2400GE w/ Radeon Vega Graphics
        Voltage: 1.4 V
        External Clock: 100 MHz
        Max Speed: 3800 MHz
        Current Speed: 3200 MHz
        Status: Populated, Enabled
        Upgrade: Socket AM4
        L1 Cache Handle: 0x002C
        L2 Cache Handle: 0x002D
        L3 Cache Handle: 0x002E
        Serial Number: Unknown
        Asset Tag: Unknown
        Part Number: Unknown
        Core Count: 4
        Core Enabled: 4
        Thread Count: 8
        Characteristics:
                64-bit capable
                Multi-Core
                Hardware Thread
                Execute Protection
                Enhanced Virtualization
                Power/Performance Control

Handle 0x0030, DMI type 18, 23 bytes
32-bit Memory Error Information
        Type: OK
        Granularity: Unknown
        Operation: Unknown
        Vendor Syndrome: Unknown
        Memory Array Address: Unknown
        Device Address: Unknown
        Resolution: Unknown

Handle 0x0031, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x002A
        Error Information Handle: 0x0030
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 8 GB
        Form Factor: SODIMM
        Set: None
        Locator: DIMM 0
        Bank Locator: P0 CHANNEL A
        Type: DDR4
        Type Detail: Synchronous Unbuffered (Unregistered)
        Speed: 2666 MT/s
        Manufacturer: Micron Technology
        Serial Number: 1FBD5656
        Asset Tag: Not Specified
        Part Number: 8ATF1G64HZ-2G6E1
        Rank: 1
        Configured Memory Speed: 1333 MT/s
        Minimum Voltage: 1.2 V
        Maximum Voltage: 1.2 V
        Configured Voltage: 1.2 V

Handle 0x0032, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x00000000000
        Ending Address: 0x003FFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x0031
        Memory Array Mapped Address Handle: 0x002B
        Partition Row Position: Unknown
        Interleave Position: Unknown
        Interleaved Data Depth: Unknown

Handle 0x0033, DMI type 18, 23 bytes
32-bit Memory Error Information
        Type: OK
        Granularity: Unknown
        Operation: Unknown
        Vendor Syndrome: Unknown
        Memory Array Address: Unknown
        Device Address: Unknown
        Resolution: Unknown

Handle 0x0034, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x002A
        Error Information Handle: 0x0033
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 8 GB
        Form Factor: SODIMM
        Set: None
        Locator: DIMM 0
        Bank Locator: P0 CHANNEL B
        Type: DDR4
        Type Detail: Synchronous Unbuffered (Unregistered)
        Speed: 2666 MT/s
        Manufacturer: Ramaxel Technology
        Serial Number: 119B5233
        Asset Tag: Not Specified
        Part Number: RMSA3260ME78HAF-2666
        Rank: 1
        Configured Memory Speed: 1333 MT/s
        Minimum Voltage: 1.2 V
        Maximum Voltage: 1.2 V
        Configured Voltage: 1.2 V

Handle 0x0035, DMI type 20, 35 bytes
Memory Device Mapped Address
        Starting Address: 0x00000000000
        Ending Address: 0x003FFFFFFFF
        Range Size: 16 GB
        Physical Device Handle: 0x0034
        Memory Array Mapped Address Handle: 0x002B
        Partition Row Position: Unknown
        Interleave Position: Unknown
        Interleaved Data Depth: Unknown

Handle 0x0036, DMI type 133, 5 bytes
OEM-specific Type
        Header and Data:
                85 05 36 00 01
        Strings:
                KHOIHGIUCCHHII

Handle 0x003B, DMI type 13, 22 bytes
BIOS Language Information
        Language Description Format: Long
        Installable Languages: 3
                en|US|iso8859-1
                fr|FR|iso8859-1
                zh|TW|unicode
        Currently Installed Language: en|US|iso8859-1

Handle 0x003C, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1500
        Internal Connector Type: None
        External Reference Designator: USB 3.0
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x003D, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1501
        Internal Connector Type: None
        External Reference Designator: USB 3.0
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x003E, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1502
        Internal Connector Type: None
        External Reference Designator: USB-C
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x003F, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1503
        Internal Connector Type: None
        External Reference Designator: USB 3.0
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x0040, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1504
        Internal Connector Type: None
        External Reference Designator: USB 3.1
        External Connector Type: Access Bus (USB)
        Port Type: USB

Handle 0x0041, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1503
        Internal Connector Type: None
        External Reference Designator: Network
        External Connector Type: RJ-45
        Port Type: Network Port

Handle 0x0042, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1704
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: Sata Express
        External Connector Type: None
        Port Type: SATA

Handle 0x0043, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1705
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: Sata Express
        External Connector Type: None
        Port Type: SATA

Handle 0x0044, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1701
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: iSATA
        External Connector Type: None
        Port Type: SATA

Handle 0x0045, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1702
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: iSATA
        External Connector Type: None
        Port Type: SATA

Handle 0x0046, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1703
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: iSATA
        External Connector Type: None
        Port Type: SATA

Handle 0x0047, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1706
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: iSATA
        External Connector Type: None
        Port Type: SATA

Handle 0x0048, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1100
        Internal Connector Type: None
        External Reference Designator: HDMI
        External Connector Type: None
        Port Type: Video Port

Handle 0x0049, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1101
        Internal Connector Type: None
        External Reference Designator: HDMI
        External Connector Type: None
        Port Type: Video Port

Handle 0x004A, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1102
        Internal Connector Type: None
        External Reference Designator: DP
        External Connector Type: None
        Port Type: Video Port

Handle 0x004B, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2100
        Internal Connector Type: None
        External Reference Designator: Front Audio
        External Connector Type: Mini Jack (headphones)
        Port Type: Audio Port

Handle 0x004C, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J2101
        Internal Connector Type: None
        External Reference Designator: Audio Jack
        External Connector Type: Mini Jack (headphones)
        Port Type: Audio Port

Handle 0x004D, DMI type 8, 9 bytes
Port Connector Information
        Internal Reference Designator: J1700
        Internal Connector Type: SAS/SATA Plug Receptacle
        External Reference Designator: Sata Express
        External Connector Type: None
        Port Type: SATA

Handle 0x004E, DMI type 9, 17 bytes
System Slot Information
        Designation: J10
        Type: PCI Express x16
        Data Bus Width: 8x or x8
        Current Usage: Available
        Length: Short
        ID: 48
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x004F, DMI type 9, 17 bytes
System Slot Information
        Designation: J3600 Pcie x8 slot
        Type: PCI Express x8
        Data Bus Width: 8x or x8
        Current Usage: Other
        Length: Short
        ID: 32
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0050, DMI type 9, 17 bytes
System Slot Information
        Designation: J3707 Pcie x4 slot
        Type: PCI Express x4
        Data Bus Width: 4x or x4
        Current Usage: Other
        Length: Short
        ID: 33
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0051, DMI type 9, 17 bytes
System Slot Information
        Designation: J3700 PCIE x4 slot from Promontory
        Type: PCI Express x4
        Data Bus Width: 4x or x4
        Current Usage: Available
        Length: Short
        ID: 20
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0052, DMI type 9, 17 bytes
System Slot Information
        Designation: J3702 PCIE x1 slot from Promontory
        Type: PCI Express x1
        Data Bus Width: 1x or x1
        Current Usage: Available
        Length: Short
        ID: 19
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0053, DMI type 9, 17 bytes
System Slot Information
        Designation: J3703 PCIE x1 slot from Promontory
        Type: PCI Express x1
        Data Bus Width: 1x or x1
        Current Usage: Available
        Length: Short
        ID: 18
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0054, DMI type 9, 17 bytes
System Slot Information
        Designation: J3701 M.2 WLAN/BT slot
        Type: PCI Express x1
        Data Bus Width: 1x or x1
        Current Usage: Available
        Length: Short
        ID: 17
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0055, DMI type 9, 17 bytes
System Slot Information
        Designation: U3700 M.2 Slot
        Type: PCI Express x1
        Data Bus Width: 1x or x1
        Current Usage: Available
        Length: Short
        ID: 53
        Characteristics:
                3.3 V is provided
                PME signal is supported
        Bus Address: 0000:00:1f.7

Handle 0x0056, DMI type 41, 11 bytes
Onboard Device
        Reference Designation: Broadcom 5762
        Type: Ethernet
        Status: Enabled
        Type Instance: 1
        Bus Address: 0000:01:00.2

Handle 0x0057, DMI type 41, 11 bytes
Onboard Device
        Reference Designation: Realtek ALC898
        Type: Sound
        Status: Enabled
        Type Instance: 1
        Bus Address: 0000:04:00.6

Handle 0x0058, DMI type 131, 22 bytes
ThinkVantage Technologies
        Version: 1
        Diagnostics: Available

Handle 0x0059, DMI type 15, 73 bytes
System Event Log
        Area Length: 4096 bytes
        Header Start Offset: 0x0000
        Header Length: 16 bytes
        Data Start Offset: 0x0010
        Access Method: Memory-mapped physical 32-bit address
        Access Address: 0x7C588018
        Status: Valid, Not Full
        Change Token: 0x00000003
        Header Format: Type 1
        Supported Log Type Descriptors: 25
        Descriptor 1: Single-bit ECC memory error
        Data Format 1: Multiple-event handle
        Descriptor 2: Multi-bit ECC memory error
        Data Format 2: Multiple-event handle
        Descriptor 3: Parity memory error
        Data Format 3: None
        Descriptor 4: Bus timeout
        Data Format 4: None
        Descriptor 5: I/O channel block
        Data Format 5: None
        Descriptor 6: Software NMI
        Data Format 6: None
        Descriptor 7: POST memory resize
        Data Format 7: None
        Descriptor 8: POST error
        Data Format 8: POST results bitmap
        Descriptor 9: PCI parity error
        Data Format 9: Multiple-event handle
        Descriptor 10: PCI system error
        Data Format 10: Multiple-event handle
        Descriptor 11: CPU failure
        Data Format 11: None
        Descriptor 12: EISA failsafe timer timeout
        Data Format 12: None
        Descriptor 13: Correctable memory log disabled
        Data Format 13: None
        Descriptor 14: Logging disabled
        Data Format 14: None
        Descriptor 15: System limit exceeded
        Data Format 15: None
        Descriptor 16: Asynchronous hardware timer expired
        Data Format 16: None
        Descriptor 17: System configuration information
        Data Format 17: None
        Descriptor 18: Hard disk information
        Data Format 18: None
        Descriptor 19: System reconfigured
        Data Format 19: None
        Descriptor 20: Uncorrectable CPU-complex error
        Data Format 20: None
        Descriptor 21: Log area reset/cleared
        Data Format 21: None
        Descriptor 22: System boot
        Data Format 22: None
        Descriptor 23: End of log
        Data Format 23: None
        Descriptor 24: OEM-specific
        Data Format 24: OEM-specific
        Descriptor 25: OEM-specific
        Data Format 25: OEM-specific

Handle 0x005A, DMI type 127, 4 bytes
End Of Table
```

## lshw

```txt
[liam@m715q-0 ~]$ sudo lshw
m715q-0.lab.wporter.org
    description: Mini PC
    product: 10VHS1P100 (LENOVO_MT_10VH_BU_Think_FM_ThinkCentre M715q)
    vendor: LENOVO
    version: ThinkCentre M715q
    serial: MJ08LQZS
    width: 64 bits
    capabilities: smbios-3.1.1 dmi-3.1.1 smp vsyscall32
    configuration: administrator_password=disabled boot=normal chassis=mini family=ThinkCentre M715q keyboard_password=enabled power-on_password=disabled sku=LENOVO_MT_10VH_BU_Think_FM_ThinkCentre M715q uuid=7cc43000-5a26-11e9-91e9-00db294e1b00
  *-core
       description: Motherboard
       product: 3130
       vendor: LENOVO
       physical id: 0
       version: SDK0J40697 WIN 3305189113242
       slot: Default string
     *-firmware
          description: BIOS
          vendor: LENOVO
          physical id: 0
          version: M1XKT39A
          date: 04/08/2019
          size: 64KiB
          capacity: 16MiB
          capabilities: pci upgrade shadowing cdboot bootselect socketedrom edd int13floppy1200 int13floppy720 int13floppy2880 int5printscreen int14serial int17printer acpi usb biosbootspecification uefi
     *-memory
          description: System Memory
          physical id: 2a
          slot: System board or motherboard
          size: 16GiB
        *-bank:0
             description: SODIMM DDR4 Synchronous Unbuffered (Unregistered) 2666 MHz (0.4 ns)
             product: 8ATF1G64HZ-2G6E1
             vendor: Micron Technology
             physical id: 0
             serial: 1FBD5656
             slot: DIMM 0
             size: 8GiB
             width: 64 bits
             clock: 2666MHz (0.4ns)
        *-bank:1
             description: SODIMM DDR4 Synchronous Unbuffered (Unregistered) 2666 MHz (0.4 ns)
             product: RMSA3260ME78HAF-2666
             vendor: Ramaxel Technology
             physical id: 1
             serial: 119B5233
             slot: DIMM 0
             size: 8GiB
             width: 64 bits
             clock: 2666MHz (0.4ns)
     *-cache:0
          description: L1 cache
          physical id: 2c
          slot: L1 - Cache
          size: 384KiB
          capacity: 384KiB
          clock: 1GHz (1.0ns)
          capabilities: pipeline-burst internal write-back unified
          configuration: level=1
     *-cache:1
          description: L2 cache
          physical id: 2d
          slot: L2 - Cache
          size: 2MiB
          capacity: 2MiB
          clock: 1GHz (1.0ns)
          capabilities: pipeline-burst internal write-back unified
          configuration: level=2
     *-cache:2
          description: L3 cache
          physical id: 2e
          slot: L3 - Cache
          size: 4MiB
          capacity: 4MiB
          clock: 1GHz (1.0ns)
          capabilities: pipeline-burst internal write-back unified
          configuration: level=3
     *-cpu
          description: CPU
          product: AMD Ryzen 5 PRO 2400GE w/ Radeon Vega Graphics
          vendor: Advanced Micro Devices [AMD]
          physical id: 2f
          bus info: cpu@0
          version: 23.17.0
          serial: Unknown
          slot: AM4
          size: 2528MHz
          capacity: 3800MHz
          width: 64 bits
          clock: 100MHz
          capabilities: lm fpu fpu_exception wp vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp x86-64 constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf rapl pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb hw_pstate ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt sha_ni xsaveopt xsavec xgetbv1 clzero irperf xsaveerptr arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif overflow_recov succor smca sev sev_es cpufreq
          configuration: cores=4 enabledcores=4 microcode=135270407 threads=8
     *-pci:0
          description: Host bridge
          product: Raven/Raven2 Root Complex
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 100
          bus info: pci@0000:00:00.0
          version: 00
          width: 32 bits
          clock: 33MHz
        *-generic UNCLAIMED
             description: IOMMU
             product: Raven/Raven2 IOMMU
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 0.2
             bus info: pci@0000:00:00.2
             version: 00
             width: 32 bits
             clock: 33MHz
             capabilities: msi ht bus_master cap_list
             configuration: latency=0
        *-pci:0
             description: PCI bridge
             product: Raven/Raven2 PCIe GPP Bridge [6:0]
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 1.2
             bus info: pci@0000:00:01.2
             version: 00
             width: 32 bits
             clock: 33MHz
             capabilities: pci pm pciexpress msi ht normal_decode bus_master cap_list
             configuration: driver=pcieport
             resources: irq:26 ioport:f000(size=4096) memory:fe900000-fe9fffff
           *-network
                description: Ethernet interface
                product: RTL8111/8168/8211/8411 PCI Express Gigabit Ethernet Controller
                vendor: Realtek Semiconductor Co., Ltd.
                physical id: 0
                bus info: pci@0000:01:00.0
                logical name: enp1s0f0
                version: 0e
                serial: 6c:4b:90:a2:7b:db
                size: 1Gbit/s
                capacity: 1Gbit/s
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress msix vpd bus_master cap_list ethernet physical tp mii 10bt 10bt-fd 100bt 100bt-fd 1000bt-fd autonegotiation
                configuration: autonegotiation=on broadcast=yes driver=r8169 driverversion=5.14.0-503.11.1.el9_5.x86_64 duplex=full ip=192.168.77.147 latency=0 link=yes multicast=yes port=twisted pair speed=1Gbit/s
                resources: irq:56 ioport:fc00(size=256) memory:fe918000-fe918fff memory:fe910000-fe913fff
           *-communication:0
                description: Serial controller
                product: RTL8111xP UART #1
                vendor: Realtek Semiconductor Co., Ltd.
                physical id: 0.1
                bus info: pci@0000:01:00.1
                version: 0e
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress msix vpd 16550 cap_list
                configuration: driver=serial latency=0
                resources: irq:31 ioport:f800(size=256) memory:fe917000-fe917fff memory:fe90c000-fe90ffff
           *-communication:1
                description: Serial controller
                product: RTL8111xP UART #2
                vendor: Realtek Semiconductor Co., Ltd.
                physical id: 0.2
                bus info: pci@0000:01:00.2
                version: 0e
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress msix vpd 16550 cap_list
                configuration: driver=serial latency=0
                resources: irq:32 ioport:f400(size=256) memory:fe916000-fe916fff memory:fe908000-fe90bfff
           *-serial UNCLAIMED
                description: IPMI Interface
                product: RTL8111xP IPMI interface
                vendor: Realtek Semiconductor Co., Ltd.
                physical id: 0.3
                bus info: pci@0000:01:00.3
                version: 0e
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress msix vpd kcs cap_list
                configuration: latency=0
                resources: ioport:f000(size=256) memory:fe915000-fe915fff memory:fe904000-fe907fff
           *-usb
                description: USB controller
                product: RTL811x EHCI host controller
                vendor: Realtek Semiconductor Co., Ltd.
                physical id: 0.4
                bus info: pci@0000:01:00.4
                version: 0e
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress msix vpd ehci bus_master cap_list
                configuration: driver=ehci-pci latency=0
                resources: irq:35 memory:fe914000-fe914fff memory:fe900000-fe903fff
              *-usbhost
                   product: EHCI Host Controller
                   vendor: Linux 5.14.0-503.11.1.el9_5.x86_64 ehci_hcd
                   physical id: 1
                   bus info: usb@3
                   logical name: usb3
                   version: 5.14
                   capabilities: usb-2.00
                   configuration: driver=hub slots=1 speed=480Mbit/s
        *-pci:1
             description: PCI bridge
             product: Raven/Raven2 PCIe GPP Bridge [6:0]
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 1.3
             bus info: pci@0000:00:01.3
             version: 00
             width: 32 bits
             clock: 33MHz
             capabilities: pci pm pciexpress msi ht normal_decode bus_master cap_list
             configuration: driver=pcieport
             resources: irq:27 memory:fe800000-fe8fffff
           *-network DISABLED
                description: Ethernet interface
                product: Wi-Fi 6 AX200
                vendor: Intel Corporation
                physical id: 0
                bus info: pci@0000:02:00.0
                logical name: wlp2s0
                version: 1a
                serial: 6c:6a:77:a4:59:cf
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress msix bus_master cap_list ethernet physical
                configuration: broadcast=yes driver=iwlwifi driverversion=5.14.0-503.11.1.el9_5.x86_64 firmware=77.85be44d3.0 cc-a0-77.ucode latency=0 link=no multicast=yes
                resources: irq:77 memory:fe800000-fe803fff
        *-pci:2
             description: PCI bridge
             product: Raven/Raven2 PCIe GPP Bridge [6:0]
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 1.4
             bus info: pci@0000:00:01.4
             version: 00
             width: 32 bits
             clock: 33MHz
             capabilities: pci pm pciexpress msi ht normal_decode bus_master cap_list
             configuration: driver=pcieport
             resources: irq:28 memory:fe700000-fe7fffff
           *-nvme
                description: NVMe device
                product: HFM128GD3JX016N
                vendor: SK hynix
                physical id: 0
                bus info: pci@0000:03:00.0
                logical name: /dev/nvme0
                version: 41020C20
                serial: CYA6N092910407724
                width: 64 bits
                clock: 33MHz
                capabilities: nvme pm msi msix pciexpress nvm_express bus_master cap_list
                configuration: driver=nvme latency=0 nqn=nqn.2021-06.com.skhynix:nvme:nvm-subsystem-sn-CYA6N092910407724 state=live
                resources: irq:59 memory:fe700000-fe703fff memory:fe705000-fe705fff memory:fe704000-fe704fff
              *-namespace:0
                   description: NVMe disk
                   physical id: 0
                   logical name: /dev/ng0n1
              *-namespace:1
                   description: NVMe disk
                   physical id: 1
                   bus info: nvme@0:1
                   logical name: /dev/nvme0n1
                   size: 119GiB (128GB)
                   capabilities: gpt-1.00 partitioned partitioned:gpt
                   configuration: guid=182f18a1-e39f-4a66-9ba8-2cc11754038f logicalsectorsize=512 sectorsize=512 wwid=nvme.1c5c-435941364e303932393130343037373234-48464d3132384744334a583031364e-00000001
                 *-volume:0
                      description: Windows FAT volume
                      vendor: mkfs.fat
                      physical id: 1
                      bus info: nvme@0:1,1
                      logical name: /dev/nvme0n1p1
                      logical name: /boot/efi
                      version: FAT16
                      serial: 87e1-7e5d
                      size: 62MiB
                      capacity: 63MiB
                      capabilities: boot fat initialized
                      configuration: FATs=2 filesystem=fat mount.fstype=vfat mount.options=rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=ascii,shortname=winnt,errors=remount-ro name=EFI System Partition state=mounted
                 *-volume:1
                      description: EXT4 volume
                      vendor: Linux
                      physical id: 2
                      bus info: nvme@0:1,2
                      logical name: /dev/nvme0n1p2
                      logical name: /boot
                      version: 1.0
                      serial: 465db050-bb06-4f2f-8da0-05c8fd5153f8
                      size: 5GiB
                      capabilities: journaled extended_attributes large_files huge_files dir_nlink recover 64bit extents ext4 ext2 initialized
                      configuration: created=2025-02-16 17:36:40 filesystem=ext4 lastmountpoint=/boot modified=2025-02-16 17:54:27 mount.fstype=ext4 mount.options=rw,seclabel,relatime mounted=2025-02-16 17:44:26 state=mounted
                 *-volume:2
                      description: LVM Physical Volume
                      vendor: Linux
                      physical id: 3
                      bus info: nvme@0:1,3
                      logical name: /dev/nvme0n1p3
                      serial: kLTAnm-Vn2p-Schp-swBz-9y1m-5YtT-ygsGDE
                      size: 114GiB
                      capabilities: multi lvm2
        *-pci:3
             description: PCI bridge
             product: Raven/Raven2 Internal PCIe GPP Bridge 0 to Bus A
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 8.1
             bus info: pci@0000:00:08.1
             version: 00
             width: 32 bits
             clock: 33MHz
             capabilities: pci pm pciexpress msi normal_decode bus_master cap_list
             configuration: driver=pcieport
             resources: irq:29 ioport:e000(size=4096) memory:fe200000-fe5fffff ioport:e0000000(size=270532608)
           *-display
                description: VGA compatible controller
                product: Raven Ridge [Radeon Vega Series / Radeon Vega Mobile Series]
                vendor: Advanced Micro Devices, Inc. [AMD/ATI]
                physical id: 0
                bus info: pci@0000:04:00.0
                logical name: /dev/fb0
                version: d6
                width: 64 bits
                clock: 33MHz
                capabilities: pm pciexpress msi msix vga_controller bus_master cap_list fb
                configuration: depth=32 driver=amdgpu latency=0 resolution=1280,1024
                resources: irq:44 memory:e0000000-efffffff memory:f0000000-f01fffff ioport:e000(size=256) memory:fe500000-fe57ffff
           *-multimedia:0
                description: Audio device
                product: Raven/Raven2/Fenghuang HDMI/DP Audio Controller
                vendor: Advanced Micro Devices, Inc. [AMD/ATI]
                physical id: 0.1
                bus info: pci@0000:04:00.1
                logical name: card0
                logical name: /dev/snd/controlC0
                logical name: /dev/snd/hwC0D0
                logical name: /dev/snd/pcmC0D3p
                logical name: /dev/snd/pcmC0D7p
                logical name: /dev/snd/pcmC0D8p
                logical name: /dev/snd/pcmC0D9p
                version: 00
                width: 32 bits
                clock: 33MHz
                capabilities: pm pciexpress msi bus_master cap_list
                configuration: driver=snd_hda_intel latency=0
                resources: irq:89 memory:fe5c8000-fe5cbfff
              *-input:0
                   product: HD-Audio Generic HDMI/DP,pcm=3
                   physical id: 0
                   logical name: input13
                   logical name: /dev/input/event10
              *-input:1
                   product: HD-Audio Generic HDMI/DP,pcm=7
                   physical id: 1
                   logical name: input14
                   logical name: /dev/input/event11
              *-input:2
                   product: HD-Audio Generic HDMI/DP,pcm=8
                   physical id: 2
                   logical name: input15
                   logical name: /dev/input/event12
              *-input:3
                   product: HD-Audio Generic HDMI/DP,pcm=9
                   physical id: 3
                   logical name: input16
                   logical name: /dev/input/event13
           *-generic
                description: Encryption controller
                product: Family 17h (Models 10h-1fh) Platform Security Processor
                vendor: Advanced Micro Devices, Inc. [AMD]
                physical id: 0.2
                bus info: pci@0000:04:00.2
                version: 00
                width: 32 bits
                clock: 33MHz
                capabilities: pm pciexpress msi msix bus_master cap_list
                configuration: driver=ccp latency=0
                resources: irq:53 memory:fe400000-fe4fffff memory:fe5cc000-fe5cdfff
           *-usb:0
                description: USB controller
                product: Raven USB 3.1
                vendor: Advanced Micro Devices, Inc. [AMD]
                physical id: 0.3
                bus info: pci@0000:04:00.3
                version: 00
                width: 64 bits
                clock: 33MHz
                capabilities: pm pciexpress msi msix xhci bus_master cap_list
                configuration: driver=xhci_hcd latency=0
                resources: irq:33 memory:fe300000-fe3fffff
              *-usbhost:0
                   product: xHCI Host Controller
                   vendor: Linux 5.14.0-503.11.1.el9_5.x86_64 xhci-hcd
                   physical id: 0
                   bus info: usb@1
                   logical name: usb1
                   version: 5.14
                   capabilities: usb-2.00
                   configuration: driver=hub slots=4 speed=480Mbit/s
                 *-usb
                      description: Keyboard
                      product: 2.4G Mouse System Control
                      vendor: SHARKOON Technologies GmbH
                      physical id: 2
                      bus info: usb@1:2
                      logical name: input4
                      logical name: /dev/input/event2
                      logical name: input4::capslock
                      logical name: input4::compose
                      logical name: input4::kana
                      logical name: input4::numlock
                      logical name: input4::scrolllock
                      logical name: input5
                      logical name: /dev/input/event3
                      logical name: input6
                      logical name: /dev/input/event4
                      logical name: /dev/input/mouse0
                      logical name: input7
                      logical name: /dev/input/event5
                      logical name: input8
                      logical name: /dev/input/event6
                      version: 2.00
                      capabilities: usb-1.10 usb
                      configuration: driver=usbhid maxpower=100mA speed=12Mbit/s
              *-usbhost:1
                   product: xHCI Host Controller
                   vendor: Linux 5.14.0-503.11.1.el9_5.x86_64 xhci-hcd
                   physical id: 1
                   bus info: usb@2
                   logical name: usb2
                   version: 5.14
                   capabilities: usb-3.10
                   configuration: driver=hub slots=4 speed=10000Mbit/s
                 *-usb
                      description: Mass storage device
                      product: RTL9210-VB
                      vendor: Realtek
                      physical id: 4
                      bus info: usb@2:4
                      logical name: scsi1
                      version: f0.01
                      serial: 012345679015
                      capabilities: usb-3.20 scsi
                      configuration: driver=uas maxpower=896mA speed=5000Mbit/s
                    *-disk
                         description: SCSI Disk
                         product: Me SK hynix 256G
                         vendor: BC901 NV
                         physical id: 0.0.0
                         bus info: scsi@1:0.0.0
                         logical name: /dev/sda
                         version: 1.00
                         serial: 0000000000000000
                         size: 238GiB (256GB)
                         capabilities: partitioned partitioned:dos
                         configuration: ansiversion=6 logicalsectorsize=512 sectorsize=512 signature=a0ae9533
                       *-volume:0
                            description: HPFS/NTFS partition
                            physical id: 1
                            bus info: scsi@1:0.0.0,1
                            logical name: /dev/sda1
                            capacity: 238GiB
                            capabilities: primary bootable
                       *-volume:1
                            description: Windows FAT volume
                            vendor: mkfs.fat
                            physical id: 2
                            bus info: scsi@1:0.0.0,2
                            logical name: /dev/sda2
                            version: FAT16
                            serial: 223c-f3f8
                            size: 31MiB
                            capacity: 32MiB
                            capabilities: primary boot fat initialized
                            configuration: FATs=2 filesystem=fat label=VTOYEFI
           *-usb:1
                description: USB controller
                product: Raven USB 3.1
                vendor: Advanced Micro Devices, Inc. [AMD]
                physical id: 0.4
                bus info: pci@0000:04:00.4
                version: 00
                width: 64 bits
                clock: 33MHz
                capabilities: pm pciexpress msi msix xhci bus_master cap_list
                configuration: driver=xhci_hcd latency=0
                resources: irq:44 memory:fe200000-fe2fffff
              *-usbhost:0
                   product: xHCI Host Controller
                   vendor: Linux 5.14.0-503.11.1.el9_5.x86_64 xhci-hcd
                   physical id: 0
                   bus info: usb@4
                   logical name: usb4
                   version: 5.14
                   capabilities: usb-2.00
                   configuration: driver=hub slots=1 speed=480Mbit/s
                 *-usb
                      description: USB hub
                      product: USB2.0 Hub
                      vendor: Genesys Logic, Inc.
                      physical id: 1
                      bus info: usb@4:1
                      version: 60.52
                      capabilities: usb-2.00
                      configuration: driver=hub maxpower=100mA slots=4 speed=480Mbit/s
                    *-usb:0
                         description: Keyboard
                         product: Cherry GmbH CHERRY Corded Device
                         vendor: Cherry GmbH
                         physical id: 2
                         bus info: usb@4:1.2
                         logical name: input10
                         logical name: /dev/input/event7
                         logical name: input10::capslock
                         logical name: input10::numlock
                         logical name: input10::scrolllock
                         logical name: input11
                         logical name: /dev/input/event8
                         version: 4.10
                         capabilities: usb-2.00 usb
                         configuration: driver=usbhid maxpower=100mA speed=2Mbit/s
                    *-usb:1
                         description: Bluetooth wireless interface
                         product: AX200 Bluetooth
                         vendor: Intel Corp.
                         physical id: 4
                         bus info: usb@4:1.4
                         version: 0.01
                         capabilities: bluetooth usb-2.01
                         configuration: driver=btusb maxpower=100mA speed=12Mbit/s
              *-usbhost:1
                   product: xHCI Host Controller
                   vendor: Linux 5.14.0-503.11.1.el9_5.x86_64 xhci-hcd
                   physical id: 1
                   bus info: usb@5
                   logical name: usb5
                   version: 5.14
                   capabilities: usb-3.10
                   configuration: driver=hub slots=1 speed=10000Mbit/s
           *-multimedia:1 UNCLAIMED
                description: Multimedia controller
                product: ACP/ACP3X/ACP6x Audio Coprocessor
                vendor: Advanced Micro Devices, Inc. [AMD]
                physical id: 0.5
                bus info: pci@0000:04:00.5
                version: 00
                width: 32 bits
                clock: 33MHz
                capabilities: pm pciexpress msi cap_list
                configuration: latency=0
                resources: memory:fe580000-fe5bffff
           *-multimedia:2
                description: Audio device
                product: Family 17h/19h HD Audio Controller
                vendor: Advanced Micro Devices, Inc. [AMD]
                physical id: 0.6
                bus info: pci@0000:04:00.6
                logical name: card1
                logical name: /dev/snd/controlC1
                logical name: /dev/snd/hwC1D0
                logical name: /dev/snd/pcmC1D0c
                logical name: /dev/snd/pcmC1D0p
                version: 00
                width: 32 bits
                clock: 33MHz
                capabilities: pm pciexpress msi bus_master cap_list
                configuration: driver=snd_hda_intel latency=0
                resources: irq:90 memory:fe5c0000-fe5c7fff
              *-input:0
                   product: HD-Audio Generic Mic
                   physical id: 0
                   logical name: input17
                   logical name: /dev/input/event14
              *-input:1
                   product: HD-Audio Generic Front Mic
                   physical id: 1
                   logical name: input18
                   logical name: /dev/input/event15
              *-input:2
                   product: HD-Audio Generic Line Out
                   physical id: 2
                   logical name: input19
                   logical name: /dev/input/event16
              *-input:3
                   product: HD-Audio Generic Front Headphone
                   physical id: 3
                   logical name: input20
                   logical name: /dev/input/event17
        *-pci:4
             description: PCI bridge
             product: Raven/Raven2 Internal PCIe GPP Bridge 0 to Bus B
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 8.2
             bus info: pci@0000:00:08.2
             version: 00
             width: 32 bits
             clock: 33MHz
             capabilities: pci pm pciexpress msi normal_decode bus_master cap_list
             configuration: driver=pcieport
             resources: irq:30 memory:fe600000-fe6fffff
           *-sata
                description: SATA controller
                product: FCH SATA Controller [AHCI mode]
                vendor: Advanced Micro Devices, Inc. [AMD]
                physical id: 0
                bus info: pci@0000:05:00.0
                version: 61
                width: 32 bits
                clock: 33MHz
                capabilities: sata pm pciexpress msi ahci_1.0 bus_master cap_list
                configuration: driver=ahci latency=0
                resources: irq:58 memory:fe600000-fe6007ff
        *-serial
             description: SMBus
             product: FCH SMBus Controller
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 14
             bus info: pci@0000:00:14.0
             version: 61
             width: 32 bits
             clock: 66MHz
             configuration: driver=piix4_smbus latency=0
             resources: irq:0
        *-isa
             description: ISA bridge
             product: FCH LPC Bridge
             vendor: Advanced Micro Devices, Inc. [AMD]
             physical id: 14.3
             bus info: pci@0000:00:14.3
             version: 51
             width: 32 bits
             clock: 66MHz
             capabilities: isa bus_master
             configuration: latency=0
           *-pnp00:00
                product: PnP device PNP0c01
                physical id: 0
                capabilities: pnp
                configuration: driver=system
           *-pnp00:01
                product: PnP device PNP0c02
                physical id: 1
                capabilities: pnp
                configuration: driver=system
           *-pnp00:02
                product: PnP device PNP0b00
                physical id: 2
                capabilities: pnp
                configuration: driver=rtc_cmos
           *-pnp00:03
                product: PnP device PNP0c02
                physical id: 3
                capabilities: pnp
                configuration: driver=system
           *-pnp00:04
                product: PnP device PNP0c02
                physical id: 4
                capabilities: pnp
                configuration: driver=system
     *-pci:1
          description: Host bridge
          product: Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 101
          bus info: pci@0000:00:01.0
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:2
          description: Host bridge
          product: Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 102
          bus info: pci@0000:00:08.0
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:3
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 0
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 103
          bus info: pci@0000:00:18.0
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:4
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 1
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 104
          bus info: pci@0000:00:18.1
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:5
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 2
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 105
          bus info: pci@0000:00:18.2
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:6
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 3
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 106
          bus info: pci@0000:00:18.3
          version: 00
          width: 32 bits
          clock: 33MHz
          configuration: driver=k10temp
          resources: irq:0
     *-pci:7
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 4
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 107
          bus info: pci@0000:00:18.4
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:8
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 5
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 108
          bus info: pci@0000:00:18.5
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:9
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 6
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 109
          bus info: pci@0000:00:18.6
          version: 00
          width: 32 bits
          clock: 33MHz
     *-pci:10
          description: Host bridge
          product: Raven/Raven2 Device 24: Function 7
          vendor: Advanced Micro Devices, Inc. [AMD]
          physical id: 10a
          bus info: pci@0000:00:18.7
          version: 00
          width: 32 bits
          clock: 33MHz
  *-input:0
       product: Power Button
       physical id: 1
       logical name: input0
       logical name: /dev/input/event0
       capabilities: platform
  *-input:1
       product: Power Button
       physical id: 2
       logical name: input1
       logical name: /dev/input/event1
       capabilities: platform
  *-input:2
       product: PC Speaker
       physical id: 3
       logical name: input12
       logical name: /dev/input/event9
       capabilities: isa
```