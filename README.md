# Sonoff GK-200MP2-B Dump

Before we start, this is my first Github repo :) So if things are off or not right looking go yell at me and I'll fix it!

I'll provide as much info here as possible so you guys can hopefully work out a way to make this camera cloudless! It's a nice (and cheap) little piece of hardware that could be perfect for a modded firmware to simply make it an ONFIV compatible camera with PTZ controlls. As of now it only provides a somewhat low bitrate RTSP stream with just 10FPS.

# Firmware Dumps
If you just want to get started without modifying the camera on your own I've provided a factory reset firmware dump image above, just browse through the files! It's a dump of the whole Flash.  
The files have been dumped after the very first bootup of the camera until it asks for "*Please use mobile phone for setup*", then rebooted and dumped with the following commands:
```
# sf read 0xc1000000 0x0 0x800000
# tftpput 0xc1000000 0x800000 ALL_VerXXXX.bin
```

# How to get shell access
You need to connect via serial to the camera since Telnet does not accept passwordless logins. The camera is easy to take apart and there are serial connections layed out on the board, so this should be super easy :) The image shown below is from the backside.

**Baudrate:** 115200  

![Serial Layout](/images/Serial_Layout.jpg)

After connecting 3.3V the camera will try to boot up and draw a lot of current, my cheapo adapter ran pretty hot so I'd suggest not using the 3.3V your adapter provides but rather disconnect it, leave GND connected and use the normal mains adapter of the camera.

Once connected you need to quickly press "Enter" to enter U-Boot. After doing so use the following commands to get a shell.

```
# setenv bootargs console=ttySGK0 rootfstype=squashfs root=/dev/mtdblock2 init=/gm/bin/busybox mem=40M phytype=0 mtdparts=gk_flash:0x50000@0(UBOOT),0x160000@0x50000(LINUX),0x340000@0x1B0000(ROOTFS),0x90000@0x500000(USER),0x260000@0x5A0000(APP),8M@0(ALL) ash

# boot
```

You then have to manually run some commands from the "`squashfs_init`" and "`init`" file to get most of the filesystem and Busybox commands working, however I myself haven't figured out all the commands you need yet. Merge requests are welcome if you do so :)

---

# Important Infos
## Getting you started
* Telnet is open on Port 23, however you can't login. It's not permitted without a password, however the root user does not have a password defined. Same applies to Serial.
* When connecting via serial to the camera you have a 1 second window to enter U-Boot


## Bootlog
This is the bootlog after powering up and rebooting the camera once.

```
Provided as a separate file above!
```

## cat /proc/cpuinfo
```
Processor       : ARMv6-compatible processor rev 7 (v6l)
BogoMIPS        : 597.60
Features        : swp half fastmult vfp edsp java tls
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x0
CPU part        : 0xb76
CPU revision    : 7

Hardware        : Goke IPC Board
Revision        : 0000
Serial          : 0000000000000000
```

## cat /proc/meminfo
```
MemTotal:          36588 kB
MemFree:           31316 kB
Buffers:             544 kB
Cached:             1112 kB
SwapCached:            0 kB
Active:              604 kB
Inactive:           1168 kB
Active(anon):        116 kB
Inactive(anon):        0 kB
Active(file):        488 kB
Inactive(file):     1168 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:           132 kB
Mapped:              392 kB
Shmem:                 0 kB
Slab:               2420 kB
SReclaimable:        648 kB
SUnreclaim:         1772 kB
KernelStack:         200 kB
PageTables:           28 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:       18292 kB
Committed_AS:        408 kB
VmallocTotal:    2031616 kB
VmallocUsed:       58896 kB
VmallocChunk:     998880 kB
```

## cat /proc/iomem
```
c0200000-c29fffff : System RAM
  c0208000-c0581fff : Kernel code
  c059e000-c05efbe3 : Kernel data
f2000000-f2000fff : gk-sd.0
f2006000-f2008000 : musb-gk
  f2006000-f2008000 : musb-gk
f200e000-f200ffff : gk-eth.0
f3001000-f3001fff : spi.1
f3003000-f3003fff : i2c.0
f3004000-f3004fff : i2c.1
f3006000-f3006fff : gk-wdt
f3008000-f3008fff : spi.0
```

## printenv
```
arm_freq=0x00112032
baudrate=115200
bootargs=console=ttySGK0 rootfstype=squashfs root=/dev/mtdblock2 init=/squashfs_init mem=40M phytype=0 mtdparts=gk_flash:0x50000@0(UBOOT),0x160000@0x50000(LINUX),0x340000@0x1B0000(ROOTFS),0x90000@0x500000(USER),0x260000@0x5A0000(APP),8M@0(ALL)
bootcmd=sf probe 0; sf read 0xc1000000 0x50000 0x160000; bootm 0xc1000000
bootdelay=1
bootfile=zImage
bsbsize=1M
consoledev=ttySGK0
ethact=goke
ethaddr=xx:xx:xx:xx:xx:xx
fileaddr=C1000000
filesize=219000
gatewayip=11.1.5.1
hostname="gk7102s"
ipaddr=192.168.200.12
loadaddr=0xC1000000
mem=40M
netdev=eth0
netmask=255.255.255.0
nfsserver=11.1.5.19
phytype=0
rootfstype=ubi.mtd=3 rootfstype=ubifs root=ubi0:rootfs
rootpath=/opt/work
serverip=192.168.200.108
sfboot=setenv bootargs console=${consoledev},${baudrate} noinitrd mem=${mem} rw ${rootfstype} init=linuxrc ip=${ipaddr}:${serverip}:${gatewayip}:${netmask}:${hostname}:${netdev} mac=${ethaddr} phytype=${phytype};sf probe 0 0;sf read ${loadaddr} ${sfkernel} ${filesize}; bootm
sfkernel=0x50000
stderr=serial
stdin=serial
stdout=serial
tftpboot=setenv bootargs root=/dev/nfs nfsroot=${nfsserver}:${rootpath},proto=tcp,nfsvers=3,nolock ip=${ipaddr}:${serverip}:${gatewayip}:${netmask}:${hostname}:${netdev} mac=${ethaddr} phytype=${phytype} console=${consoledev},${baudrate} mem=${mem};tftpboot ${bootfile};bootm

Environment size: 1406/65532 bytes
```

## sf probe 0
```
SF:    8 MiB [page:256 Bytes] [sector:64 KiB] [count:128] (W25Q64FV)
```


# Board Images
![Board Backside](/images/Board_Back.jpg)
![Board Front](/images/Board_Front.jpg)