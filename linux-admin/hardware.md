# Hardware
## CPU
Detect P- and E-cores <br>
i7-1255U:
```
lscpu --all --extended
..
CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE    MAXMHZ   MINMHZ       MHZ
  0    0      0    0 0:0:0:0          yes 4700.0000 400.0000 1478.7679
  1    0      0    0 0:0:0:0          yes 4700.0000 400.0000  400.0000
  2    0      0    1 4:4:1:0          yes 4700.0000 400.0000  847.9270
  3    0      0    1 4:4:1:0          yes 4700.0000 400.0000  400.0000
  4    0      0    2 8:8:2:0          yes 3500.0000 400.0000  710.5360
  5    0      0    3 9:9:2:0          yes 3500.0000 400.0000 1113.3420
  6    0      0    4 10:10:2:0        yes 3500.0000 400.0000  953.9940
  7    0      0    5 11:11:2:0        yes 3500.0000 400.0000  781.3320
  8    0      0    6 12:12:3:0        yes 3500.0000 400.0000  946.2730
  9    0      0    7 13:13:3:0        yes 3500.0000 400.0000 1154.3719
 10    0      0    8 14:14:3:0        yes 3500.0000 400.0000 1114.2540
 11    0      0    9 15:15:3:0        yes 3500.0000 400.0000 1063.8230
```
- CPU 0 and CPU 1 belong to CORE 0. CORE 0 is a P-core 
- CPU 2 and CPU 3 belong to CORE 1. CORE 1 is a P-core
- Also take a look at MAXMHZ, compare it with technical specifications
