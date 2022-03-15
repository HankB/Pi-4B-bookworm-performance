## Pi 4B Bookworm performance

Pi 4B seems to run much slower on Debian Bookworm than Debian Bullseye. (This is not about the Raspberry Pi OS.)

## Benchmarks - backstory

While overclocking a Pi 4B I ran the `hardinfo` included benchmarks and was surprised to see that the results were poor when running Bookwork as compared to Bullseye (and R-Pi OS.) I've started using the `sysbench` CPU benchmark as it is quick (10s), seems to produce similar results (and does not require a GUI.) Examples are

Bullseye

```text
hbarta@kweli:~$ time sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   584.72

General statistics:
    total time:                          10.0006s
    total number of events:              5850

Latency (ms):
         min:                                    1.70
         avg:                                    1.71
         max:                                    2.99
         95th percentile:                        1.73
         sum:                                 9997.34

Threads fairness:
    events (avg/stddev):           5850.0000/0.00
    execution time (avg/stddev):   9.9973/0.00


real    0m10.022s
user    0m9.994s
sys     0m0.008s
hbarta@kweli:~$
```

Bookworm

```text
root@kweli-t:/home/hbarta# time sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   233.03

General statistics:
    total time:                          10.0038s
    total number of events:              2333

Latency (ms):
         min:                                    4.28
         avg:                                    4.29
         max:                                    4.34
         95th percentile:                        4.33
         sum:                                10001.67

Threads fairness:
    events (avg/stddev):           2333.0000/0.00
    execution time (avg/stddev):   10.0017/0.00


real    0m10.051s
user    0m10.034s
sys     0m0.016s
root@kweli-t:/home/hbarta#
```

Going forward (unless someone requests a different benchmark) I will report the "total number of events" which are 5850/Bullseye and 2333/Bookworm in this sample.

## 2022-03-14 Initial exploration


Working with diederik on https://webchat.oftc.net/?channels=debian-raspberrypi, tried the following:

Install the 5.17 kernel from experimental backports. No change.

Download and install an older kernel (*)

```text
sudo apt install \
  ./linux-headers-5.14.0-trunk-arm64_5.14-1~exp2_arm64.deb \
  ./linux-headers-5.14.0-trunk-common_5.14-1~exp2_all.deb \
  ./linux-image-5.14.0-trunk-arm64-unsigned_5.14-1~exp2_arm64.deb \
  ./linux-kbuild-5.14_5.14-1~exp2_arm64.deb
```

Locate, download and install the kernel from Bullseye.

```text
sudo apt install \
./linux-headers-5.10.0-10-arm64_5.10.84-1_arm64.deb \
./linux-headers-5.10.0-10-common_5.10.84-1_all.deb \
./linux-headers-arm64_5.10.84-1_arm64.deb \
./linux-image-arm64_5.10.84-1_arm64.deb \
./linux-kbuild-5.10_5.10.84-1_arm64.deb \
./linux-image-5.10.0-10-arm64_5.10.84-1_arm64.deb
```

(*) I had ZFS installed and this required DKMS and the kernel headers. And took longer. :-/

None of these changes improved performance. diederik suggested posting to the debian-arm mailing list so I'm collecting my notes, will perform a clean install (w/out ZFS) and start from there.

## Fresh install

Using `20220121_raspi_4_bookworm.img.xz`

```text
fell off scroll buffer
```

After `apt update;apt upgrade` and reboot (and misc like adduser set timezone etc.)

```text
hbarta@up:~$ sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   231.82

General statistics:
    total time:                          10.0042s
    total number of events:              2321

Latency (ms):
         min:                                    4.28
         avg:                                    4.31
         max:                                    5.69
         95th percentile:                        4.49
         sum:                                10001.82

Threads fairness:
    events (avg/stddev):           2321.0000/0.00
    execution time (avg/stddev):   10.0018/0.00

hbarta@up:~$
```

Edit `/boot/firmware/config.txt` to revert kernel to 5.15.

```text
hbarta@up:~$ sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   232.14

General statistics:
    total time:                          10.0030s
    total number of events:              2324

Latency (ms):
         min:                                    4.28
         avg:                                    4.30
         max:                                    5.12
         95th percentile:                        4.33
         sum:                                10000.07

Threads fairness:
    events (avg/stddev):           2324.0000/0.00
    execution time (avg/stddev):   10.0001/0.00

hbarta@up:~$ uname -a
Linux up 5.15.0-2-arm64 #1 SMP Debian 5.15.5-2 (2021-12-18) aarch64 GNU/Linux
hbarta@up:~$
```

Repeating fresh install to collect further information.

Detailed log.

`adduser hbarta`

```text
hbarta@rpi4-20220121:~$ lscpu
Architecture:            aarch64
  CPU op-mode(s):        32-bit, 64-bit
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               ARM
  Model name:            Cortex-A72
    Model:               3
    Thread(s) per core:  1
    Core(s) per cluster: 4
    Socket(s):           -
    Cluster(s):          1
    Stepping:            r0p3
    CPU max MHz:         1500.0000
    CPU min MHz:         600.0000
    BogoMIPS:            108.00
    Flags:               fp asimd evtstrm crc32 cpuid
NUMA:                    
  NUMA node(s):          1
  NUMA node0 CPU(s):     0-3
Vulnerabilities:         
  Itlb multihit:         Not affected
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Spec store bypass:     Vulnerable
  Spectre v1:            Mitigation; __user pointer sanitization
  Spectre v2:            Vulnerable
  Srbds:                 Not affected
  Tsx async abort:       Not affected
hbarta@rpi4-20220121:~$ 
```

Of note the line

```text
    CPU max MHz:         1500.0000
```

Does not appear following upgrade.

As root

```text
apt update
apt install sysbench
```

103 packages to update.

```text
root@rpi4-20220121:~# apt install sytsbench
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package sytsbench
root@rpi4-20220121:~# apt install sysbench  
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libaio1 libldap-2.4-2 libldap-common libluajit-5.1-2 libluajit-5.1-common libmariadb3 libpq5 libsasl2-2 libsasl2-modules libsasl2-modules-db mariadb-common mysql-common
Suggested packages:
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql
The following NEW packages will be installed:
  libaio1 libldap-2.4-2 libldap-common libluajit-5.1-2 libluajit-5.1-common libmariadb3 libpq5 libsasl2-2 libsasl2-modules libsasl2-modules-db mariadb-common mysql-common
  sysbench
0 upgraded, 13 newly installed, 0 to remove and 103 not upgraded.
Need to get 1379 kB of archives.
After this operation, 3824 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
...
```

`sysbench` after update and upgrade not performed.

```text
hbarta@rpi4-20220121:~$ sysbench --test=cpu --cpu-max-prime=20000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   582.83

General statistics:
    total time:                          10.0009s
    total number of events:              5831

Latency (ms):
         min:                                    1.71
         avg:                                    1.71
         max:                                    2.62
         95th percentile:                        1.73
         sum:                                 9998.72

Threads fairness:
    events (avg/stddev):           5831.0000/0.00
    execution time (avg/stddev):   9.9987/0.00

hbarta@rpi4-20220121:~$ 
```

Packages pending upgrade

```text
barta@rpi4-20220121:~$ apt list --upgradable
Listing... Done
adduser/testing 3.120 all [upgradable from: 3.118]
apparmor/testing 3.0.4-2 arm64 [upgradable from: 3.0.3-6]
apt-utils/testing 2.4.1 arm64 [upgradable from: 2.3.14]
apt/testing 2.4.1 arm64 [upgradable from: 2.3.14]
base-files/testing 12.2 arm64 [upgradable from: 12]
bsdutils/testing 1:2.37.3-1+b1 arm64 [upgradable from: 1:2.37.2-6]
ca-certificates/testing 20211016 all [upgradable from: 20210119]
dash/testing 0.5.11+git20210903+057cd650a4ed-7 arm64 [upgradable from: 0.5.11+git20210903+057cd650a4ed-3]
dbus-bin/testing 1.14.0-1 arm64 [upgradable from: 1.12.20-3]
dbus-daemon/testing 1.14.0-1 arm64 [upgradable from: 1.12.20-3]
dbus-session-bus-common/testing 1.14.0-1 all [upgradable from: 1.12.20-3]
dbus-system-bus-common/testing 1.14.0-1 all [upgradable from: 1.12.20-3]
dbus/testing 1.14.0-1 arm64 [upgradable from: 1.12.20-3]
debianutils/testing 5.7-0.1 arm64 [upgradable from: 4.11.2]
fdisk/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
findutils/testing 4.9.0-2 arm64 [upgradable from: 4.8.0-1]
gcc-10-base/testing 10.3.0-14 arm64 [upgradable from: 10.3.0-13]
gcc-11-base/testing 11.2.0-16 arm64 [upgradable from: 11.2.0-14]
ifupdown/testing 0.8.37 arm64 [upgradable from: 0.8.36+nmu1]
init-system-helpers/testing 1.62 all [upgradable from: 1.61]
init/testing 1.62 arm64 [upgradable from: 1.61]
iproute2/testing 5.16.0-2 arm64 [upgradable from: 5.16.0-1]
iputils-ping/testing 3:20211215-1 arm64 [upgradable from: 3:20210202-1]
isc-dhcp-client/testing 4.4.2-P1-1 arm64 [upgradable from: 4.4.1-2.3]
isc-dhcp-common/testing 4.4.2-P1-1 arm64 [upgradable from: 4.4.1-2.3]
klibc-utils/testing 2.0.10-4 arm64 [upgradable from: 2.0.10-3]
libapparmor1/testing 3.0.4-2 arm64 [upgradable from: 3.0.3-6]
libapt-pkg6.0/testing 2.4.1 arm64 [upgradable from: 2.3.14]
libargon2-1/testing 0~20171227-0.3 arm64 [upgradable from: 0~20171227-0.2]
libaudit-common/testing 1:3.0.7-1 all [upgradable from: 1:3.0.6-1]
libaudit1/testing 1:3.0.7-1 arm64 [upgradable from: 1:3.0.6-1+b1]
libblkid1/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
libbpf0/testing 1:0.7.0-2 arm64 [upgradable from: 1:0.5.0-1]
libbsd0/testing 0.11.5-1+b1 arm64 [upgradable from: 0.11.3-1]
libc-bin/testing 2.33-7 arm64 [upgradable from: 2.33-2]
libc6/testing 2.33-7 arm64 [upgradable from: 2.33-2]
libcrypt1/testing 1:4.4.27-1.1 arm64 [upgradable from: 1:4.4.27-1]
libdbus-1-3/testing 1.14.0-1 arm64 [upgradable from: 1.12.20-3]
libexpat1/testing 2.4.4-1 arm64 [upgradable from: 2.4.3-1]
libfdisk1/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
libffi8/testing 3.4.2-4 arm64 [upgradable from: 3.4.2-3]
libfido2-1/testing 1.10.0-1 arm64 [upgradable from: 1.9.0-2]
libgcc-s1/testing 12-20220302-1 arm64 [upgradable from: 11.2.0-14]
libgnutls30/testing 3.7.3-4+b1 arm64 [upgradable from: 3.7.2-5]
libgpg-error0/testing 1.43-3 arm64 [upgradable from: 1.43-1]
libgssapi-krb5-2/testing 1.19.2-2 arm64 [upgradable from: 1.18.3-7]
libk5crypto3/testing 1.19.2-2 arm64 [upgradable from: 1.18.3-7]
libkeyutils1/testing 1.6.1-3 arm64 [upgradable from: 1.6.1-2]
libklibc/testing 2.0.10-4 arm64 [upgradable from: 2.0.10-3]
libkrb5-3/testing 1.19.2-2 arm64 [upgradable from: 1.18.3-7]
libkrb5support0/testing 1.19.2-2 arm64 [upgradable from: 1.18.3-7]
liblocale-gettext-perl/testing 1.07-4+b2 arm64 [upgradable from: 1.07-4+b1]
libmount1/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
libncurses6/testing 6.3-2 arm64 [upgradable from: 6.3-1]
libncursesw6/testing 6.3-2 arm64 [upgradable from: 6.3-1]
libnftables1/testing 1.0.2-1 arm64 [upgradable from: 1.0.1-1]
libnl-3-200/testing 3.5.0-0.1 arm64 [upgradable from: 3.4.0-1+b1]
libnl-genl-3-200/testing 3.5.0-0.1 arm64 [upgradable from: 3.4.0-1+b1]
libnl-route-3-200/testing 3.5.0-0.1 arm64 [upgradable from: 3.4.0-1+b1]
libpam-systemd/testing 250.3-2 arm64 [upgradable from: 250.2-3]
libpcsclite1/testing 1.9.5-3 arm64 [upgradable from: 1.9.5-1]
libprocps8/testing 2:3.3.17-7+b1 arm64 [upgradable from: 2:3.3.17-6]
libselinux1/testing 3.3-1+b2 arm64 [upgradable from: 3.3-1+b1]
libsemanage2/testing 3.3-1+b2 arm64 [upgradable from: 3.3-1+b1]
libsmartcols1/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
libstdc++6/testing 12-20220302-1 arm64 [upgradable from: 11.2.0-14]
libsystemd0/testing 250.3-2 arm64 [upgradable from: 250.2-3]
libtext-charwidth-perl/testing 0.04-10+b2 arm64 [upgradable from: 0.04-10+b1]
libtext-iconv-perl/testing 1.7-7+b2 arm64 [upgradable from: 1.7-7+b1]
libtinfo6/testing 6.3-2 arm64 [upgradable from: 6.3-1]
libudev1/testing 250.3-2 arm64 [upgradable from: 250.2-3]
libunistring2/testing 1.0-1 arm64 [upgradable from: 0.9.10-6]
libuuid1/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
libxmuu1/testing 2:1.1.3-3 arm64 [upgradable from: 2:1.1.2-2+b3]
linux-base/testing 4.8 all [upgradable from: 4.7]
linux-image-arm64/testing 5.16.12-1 arm64 [upgradable from: 5.15.5-2]
login/testing 1:4.11.1+dfsg1-2 arm64 [upgradable from: 1:4.8.1-2]
logrotate/testing 3.19.0-2 arm64 [upgradable from: 3.19.0-1]
mawk/testing 1.3.4.20200120-3 arm64 [upgradable from: 1.3.4.20200120-2]
mount/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
nano/testing 6.2-1 arm64 [upgradable from: 6.0-1]
ncurses-base/testing 6.3-2 all [upgradable from: 6.3-1]
ncurses-bin/testing 6.3-2 arm64 [upgradable from: 6.3-1]
ncurses-term/testing 6.3-2 all [upgradable from: 6.3-1]
nftables/testing 1.0.2-1 arm64 [upgradable from: 1.0.1-1]
openssh-client/testing 1:8.9p1-3 arm64 [upgradable from: 1:8.7p1-4]
openssh-server/testing 1:8.9p1-3 arm64 [upgradable from: 1:8.7p1-4]
openssh-sftp-server/testing 1:8.9p1-3 arm64 [upgradable from: 1:8.7p1-4]
passwd/testing 1:4.11.1+dfsg1-2 arm64 [upgradable from: 1:4.8.1-2]
perl-base/testing 5.34.0-3 arm64 [upgradable from: 5.32.1-6]
procps/testing 2:3.3.17-7+b1 arm64 [upgradable from: 2:3.3.17-6]
raspi-firmware/testing 1.20220120+ds-1 arm64 [upgradable from: 1.20210805+ds-1]
rsyslog/testing 8.2202.0-1 arm64 [upgradable from: 8.2112.0-2]
ssh/testing 1:8.9p1-3 all [upgradable from: 1:8.7p1-4]
systemd-sysv/testing 250.3-2 arm64 [upgradable from: 250.2-3]
systemd-timesyncd/testing 250.3-2 arm64 [upgradable from: 250.2-3]
systemd/testing 250.3-2 arm64 [upgradable from: 250.2-3]
tasksel-data/testing 3.69+rebuild all [upgradable from: 3.68]
tasksel/testing 3.69+rebuild all [upgradable from: 3.68]
udev/testing 250.3-2 arm64 [upgradable from: 250.2-3]
util-linux/testing 2.37.3-1+b1 arm64 [upgradable from: 2.37.2-6]
vim-tiny/testing 2:8.2.3995-1+b2 arm64 [upgradable from: 2:8.2.3995-1]
xxd/testing 2:8.2.3995-1+b2 arm64 [upgradable from: 2:8.2.3995-1]
hbarta@rpi4-20220121:~$ 
```

## raspi-firmware update

To further track down the cause I started installing some packages that I thought might be the culprit and installed the following from the list of updates. Packages paired together were installed together and following each install I rebooted and tested using `lscpu` and `sysbench`. Following the install of `raspi-firmware` the performance regression appeared.

```text
init-system-helpers/testing 1.62 all [upgradable from: 1.61]
init/testing 1.62 arm64 [upgradable from: 1.61]

libbpf0/testing 1:0.7.0-2 arm64 [upgradable from: 1:0.5.0-1]

libprocps8/testing 2:3.3.17-7+b1 arm64 [upgradable from: 2:3.3.17-6]
procps/testing 2:3.3.17-7+b1 arm64 [upgradable from: 2:3.3.17-6]

libsystemd0/testing 250.3-2 arm64 [upgradable from: 250.2-3]
systemd/testing 250.3-2 arm64 [upgradable from: 250.2-3]

libudev1/testing 250.3-2 arm64 [upgradable from: 250.2-3]

raspi-firmware/testing 1.20220120+ds-1 arm64 [upgradable from: 1.20210805+ds-1]
```

## 2022-03-15 testing with SD card

Using the sysbench fileio test (from <https://raspberrypi.stackexchange.com/questions/3371/how-can-i-stress-test-my-raspberry-pi>)

### Clean install to SD card and with no upgrades, update and sysbench only installed.

```text
time sysbench --test=fileio --file-total-size=2G prepare
time sysbench --test=fileio --file-total-size=2G --file-test-mode=rndrw  --max-time=300 --max-requests=0 run
time sysbench --test=fileio --file-total-size=2G cleanup
```

```text
hbarta@down:~$ time sysbench --test=fileio --file-total-size=2G prepare
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

128 files, 16384Kb each, 2048Mb total
Creating files for the test...
Extra file open flags: (none)
Creating file test_file.0
Creating file test_file.1
...
Creating file test_file.126
Creating file test_file.127
2147483648 bytes written in 113.43 seconds (18.05 MiB/sec).

real    1m53.458s
user    0m0.153s
sys     0m13.913s
hbarta@down:~$


hbarta@down:~$ time sysbench --test=fileio --file-total-size=2G --file-test-mode=rndrw  --max-time=300 --max-requests=0 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
WARNING: --max-time is deprecated, use --time instead
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Extra file open flags: (none)
128 files, 16MiB each
2GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!


File operations:
    reads/s:                      6.22
    writes/s:                     4.15
    fsyncs/s:                     13.34

Throughput:
    read, MiB/s:                  0.10
    written, MiB/s:               0.06

General statistics:
    total time:                          308.4489s
    total number of events:              7188

Latency (ms):
         min:                                    0.00
         avg:                                   41.79
         max:                                 9009.86
         95th percentile:                       68.05
         sum:                               300392.74

Threads fairness:
    events (avg/stddev):           7188.0000/0.00
    execution time (avg/stddev):   300.3927/0.00


real    5m8.482s
user    0m0.027s
sys     0m0.333s
hbarta@down:~$ 

hbarta@down:~$ time sysbench --test=fileio --file-total-size=2G cleanup
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Removing test files...

real    0m0.651s
user    0m0.012s
sys     0m0.627s
hbarta@down:~$
```

Upgrade `raspi-firmware`

```text
raspi-firmware/testing 1.20220120+ds-1 arm64 [upgradable from: 1.20210805+ds-1]
```

```text
hbarta@down:~$ time sysbench --test=fileio --file-total-size=2G prepare
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

128 files, 16384Kb each, 2048Mb total
Creating files for the test...
Extra file open flags: (none)
Creating file test_file.0
Creating file test_file.1
...
Creating file test_file.126
Creating file test_file.127
2147483648 bytes written in 113.46 seconds (18.05 MiB/sec).

real    1m53.568s
user    0m0.160s
sys     0m17.705s
hbarta@down:~$ 


hbarta@down:~$ time sysbench --test=fileio --file-total-size=2G --file-test-mode=rndrw  --max-time=300 --max-requests=0 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
WARNING: --max-time is deprecated, use --time instead
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Extra file open flags: (none)
128 files, 16MiB each
2GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!


File operations:
    reads/s:                      5.73
    writes/s:                     3.82
    fsyncs/s:                     12.36

Throughput:
    read, MiB/s:                  0.09
    written, MiB/s:               0.06

General statistics:
    total time:                          303.5714s
    total number of events:              6525

Latency (ms):
         min:                                    0.01
         avg:                                   46.02
         max:                                 1880.98
         95th percentile:                      634.66
         sum:                               300250.09

Threads fairness:
    events (avg/stddev):           6525.0000/0.00
    execution time (avg/stddev):   300.2501/0.00


real    5m3.658s
user    0m0.050s
sys     0m0.291s
hbarta@down:~$ 

hbarta@down:~$ time sysbench --test=fileio --file-total-size=2G cleanup
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Removing test files...

real    0m1.648s
user    0m0.020s
sys     0m1.614s
hbarta@down:~$
```

## 2022-03-15 narrow down the offending commit

<https://github.com/raspberrypi/firmware/tags?>

Procedure (suggested by Diederik de Haas.)

* Install fresh copy of OS and update/upgrade all packages except `raspi-config`
* Reboot and confirm normal operation

* Save a copy of /boot/firmware
* Download a tag (choose by binary search/bisect)
* Extract somewhere convenient.
* Copy `bootcode.bin`,`*.dat` and `*.elf` to `/boot/firmware/`
* Reboot and test
* rinse repeat.

|tag|result|comment|
|---|---|---|
|1.20211007|good||
|1.20220118|bad||
|1.20211029|good||
|1.20211118|good||
|1.20220308|bad|most recent|
|1.20211118|good|double check|
