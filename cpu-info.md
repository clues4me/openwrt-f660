# SOME INFO ABOUT F660 HARDWARE


U-Boot 2009.08 (Sep 20 2011 - 17:28:41)

    CPU:   Feroceon (Rev 1) @ 1200Mhz - LE, L2 @ 400Mhz
        DDR3 @ 400Mhz, TClock @ 200Mhz, P/V ID=7/15
    DRAM:  128 MB
        CS 0: base 0x00000000 size 128 MB
        Addresses 26M - 0M are saved for the U-Boot usage.
    NAND:  1bit HM ECC, Size: 32 MiB
    Modules Detected:
        GPON module detected.
        TDM module.
        Ethernet Switch on MAC0.
        3xFE PHY Module.
        GE-PHY on Switch port #0.
    Net:   egiga0 [PRIME], egiga1
    Hit 1 to upgrade softwate version

Linux version 2.6.21.5 (wangkai at localhost.localdomain) (gcc version 4.3.4 
(Build
root 2010.05) ) #44 Tue Sep 20 17:53:59 CST 2011
    CPU: ARM926EJ-S [56251311] revision 1 (ARMv5TE), cr=00053977
    Machine: Feroceon-KW2
    Using UBoot passing parameters structure
    Memory policy: ECC disabled, Data cache writeback
    <7>On node 0 totalpages: 32512
    <7>  Normal zone: 0 pages used for memmap
    CPU0: D VIVT write-back cache
    CPU0: I cache: 16384 bytes, associativity 4, 32 byte lines, 128 sets
    CPU0: D cache: 16384 bytes, associativity 4, 32 byte lines, 128 sets
    Built 1 zonelists.  Total pages: 32258
Kernel command line: console=ttyS0,115200 root=/dev/ram0 rw load_ramdisk=1 
rdini
t=/sbin/init mv_net_config=0 mem=127M
PID hash table entries: 512 (order: 9, 2048 bytes)
Console: colour dummy device 80x30
Dentry cache hash table entries: 16384 (order: 4, 65536 bytes)
Inode-cache hash table entries: 8192 (order: 3, 32768 bytes)
    Memory: 127MB = 127MB total
    Memory: 116352KB available (3536K code, 714K data, 8168K init)
Mount-cache hash table entries: 512
<6>CPU: Testing write buffer coherency: ok
pdt_cspkernel_init
NET: Registered protocol family 16

CPU Interface
-------------
    SDRAM_CS0 ....base 00000000, size 128MB
    SDRAM_CS1 ....no such
    SDRAM_CS2 ....no such
    SDRAM_CS3 ....no such
    DEVICE_CS0 ....no such
    DEVICE_CS1 ....no such
    DEVICE_CS2 ....no such
    DEVICE_CS3 ....no such
    PEX0_MEM ....base f3000000, size  16MB
    PEX0_IO ....base f2000000, size   1MB
    PEX1_MEM ....base f4000000, size  16MB
    PEX1_IO ....base f2100000, size   1MB
    INTER_REGS ....base f1000000, size   1MB
    NAND_NOR_CS ....base f8000000, size   2MB
    SPI_CS0 ....base f0000000, size  16MB
    SPI_CS1 ....no such
    SPI_CS2 ....no such
    SPI_CS3 ....no such
    <4>SPI_CS4 ....no such
    SPI_CS5 ....no such
    SPI_CS6 ....no such
    SPI_CS7 ....no such
    SPI_B_CS0 ....no such
    BOOT_ROM_CS ....no such
    <4>DEV_BOOTCS ....no such
    CRYPT1_ENG ....no such
    CRYPT2_ENG ....no such
    PNC_BM ....base f5000000, size   1MB
    ETH_CTRL ....base f5100000, size   1MB
    PON_CTRL ....base f5200000, size   1MB
    NFC_CTRL ....no such

  Marvell Development Board (LSP Version 
KW2_LSP_1.0.4_p26_KERNEL_2.6.21.5_NQ)--
 ZTE-88F6560-FXXX  Soc: MV88F6560 Rev 2 LE