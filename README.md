## Pi 4B Bookworm performance

Pi 4B seems to run much slower on Debian Bookworm than Dehbian Bullseye. (This is not about the Raspberry Pi OS.)

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

None of these changes improved performance. diederik suggested posting to the kernel mailing list so I'm collecting my notes, will perform a clean install (w/out ZFS) and start from there.




