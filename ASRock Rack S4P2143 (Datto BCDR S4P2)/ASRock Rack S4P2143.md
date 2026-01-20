
# ASRock Rack S4P2143

## Fetch

```txt
liveuser@localhost-live:~$ fastfetch
             .',;::::;,'.                 liveuser@localhost-live
         .';:cccccccccccc:;,.             -----------------------
      .;cccccccccccccccccccccc;.          OS: Fedora Linux 43 (Workstation Edition) x86_64
    .:cccccccccccccccccccccccccc:.        Kernel: Linux 6.17.1-300.fc43.x86_64
  .;ccccccccccccc;.:dddl:.;ccccccc;.      Uptime: 30 mins
 .:ccccccccccccc;OWMKOOXMWd;ccccccc:.     Packages: 1982 (rpm)
.:ccccccccccccc;KMMc;cc;xMMc;ccccccc:.    Shell: bash 5.3.0
,cccccccccccccc;MMM.;cc;;WW:;cccccccc,    Display (DELL P2219H): 1920x1080 in 22", 60 Hz [External]
:cccccccccccccc;MMM.;cccccccccccccccc:    DE: GNOME 49.1
:ccccccc;oxOOOo;MMM000k.;cccccccccccc:    WM: Mutter (Wayland)
cccccc;0MMKxdd:;MMMkddc.;cccccccccccc;    WM Theme: Adwaita
ccccc;XMO';cccc;MMM.;cccccccccccccccc'    Theme: Adwaita [GTK2/3/4]
ccccc;MMo;ccccc;MMW.;ccccccccccccccc;     Icons: Adwaita [GTK2/3/4]
ccccc;0MNc.ccc.xMMd;ccccccccccccccc;      Font: Adwaita Sans (11pt) [GTK2/3/4]
cccccc;dNMWXXXWM0:;cccccccccccccc:,       Cursor: Adwaita (24px)
cccccccc;.:odl:.;cccccccccccccc:,.        Terminal: /dev/pts/1
ccccccccccccccccccccccccccccc:'.          CPU: Intel(R) Xeon(R) D-2143IT (16) @ 3.00 GHz
:ccccccccccccccccccccccc:;,..             GPU: ASPEED Technology, Inc. ASPEED Graphics Family
 ':cccccccccccccccc::;,.                  Memory: 6.87 GiB / 30.99 GiB (22%)
                                          Swap: 0 B / 8.00 GiB (0%)
                                          Disk (/): 1.11 GiB / 6.20 GiB (18%) - overlay
                                          Disk (/run/initramfs/live): 2.55 GiB / 2.55 GiB (100%) - iso9660 [Read-only]
                                          Local IP (enp181s0f0np0): 192.168.77.250/24
                                          Locale: en_US.UTF-8

```

## Introduction

This is the board out of a Datto S4P2 "BCDR" backup appliance. This was originally in a "pizza box" style 1U case. It will probably not be staying with me for all that long for reasons that will soon become apparent (power consumption!)

The CPU is a Xeon D-2143IT. This is an 8-core, 16-thread Skylake part that notably is a system on a chip (so no chipset here) with integrated Intel X722 Ethernet NICs (the OCP-style mezzanine on this board is just the PHY component). It has a reasonable amount of cache (32k L1i, 32k L1d, 1m x8 L2, 11m L3), clocks reasonably high (not really, 3 GHz 1c) and has a reasonably low TDP (65W).

The rest of the board is normal - mATX, 8 DIMM slots (2DPC), a PCI-E x16, a PCI-E x4 slot, and a single M.2 NVME slot. Onboard SATA is exposed as two SFF-8643 connectors and four standard SATA connectors (12 ports in total).

The BMC is an ASpeed AST2500 (if I recall correctly). I didn't bother setting it up because:

## Power consumption

All figures with:

- Xeon D-2143IT
- Mezzanine 2x10GBe
- Single 128gb NVME SSD
- 450w Seasonic 80+ Gold PSU (no idea what model)

From wall with a Kill-A-Watt.

Idle, in BIOS: ~75w
Idle, in F43: ~55w
Idle, in F43, post-Powertop: ~51w
GB6 multi: ~80w

Yeah. Not good.

## Geekbench 6

### Fedora Workstation 43

Single core: 781
Multi core: 5301
[Link](https://browser.geekbench.com/v6/cpu/16169327)

Pretty shabby score, too. Almost half of a normal 8th gen i5 or i7's single-core score. At many times the power.
