# Check CPU Info

This document explains how to check CPU information under Linux. If we want to know overall information, we can use `lscpu`:

```bash
test-server:[~]$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                24
On-line CPU(s) list:   0-23
Thread(s) per core:    2
Core(s) per socket:    6
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 63
Model name:            Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz
Stepping:              2
CPU MHz:               1200.468
CPU max MHz:           3200.0000
CPU min MHz:           1200.0000
BogoMIPS:              4800.83
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              15360K
NUMA node0 CPU(s):     0,2,4,6,8,10,12,14,16,18,20,22
NUMA node1 CPU(s):     1,3,5,7,9,11,13,15,17,19,21,23
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb invpcid_single ssbd ibrs ibpb stibp kaiser tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid cqm xsaveopt cqm_llc cqm_occup_llc dtherm ida arat pln pts md_clear flush_l1d
```

and if we are interested in how many sockets (physical CPUs), cores and hyperthreads we have, we can filter the result of `lscpu` by:

```
test-server:[~]$ lscpu | grep -E '^Thread|^Core|^Socket|^CPU\('
CPU(s):                24
Thread(s) per core:    2
Core(s) per socket:    6
Socket(s):             2
```

which means we have 2 sockets/physical CPUs, each of which has 6 cores and each core has two hyperthreads, so we totally have 2 * 6 * 2 = 24 processors. Each processor has a unique ID starting from 0, if we want to know detailed information about each processor, we can check the file `/proc/cpuinfo`:

```bash
test-server:[~]$ cat /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 63
model name      : Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz
stepping        : 2
microcode       : 0x43
cpu MHz         : 1200.093
cache size      : 15360 KB
physical id     : 0
siblings        : 12
core id         : 0
cpu cores       : 6
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 15
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb invpcid_single ssbd ibrs ibpb stibp kaiser tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid cqm xsaveopt cqm_llc cqm_occup_llc dtherm ida arat pln pts md_clear flush_l1d
bugs            : cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs itlb_multihit
bogomips        : 4799.68
clflush size    : 64
cache_alignment : 64
address sizes   : 46 bits physical, 48 bits virtual
power management:

...... (information about other processors)
```

in the result about, we can know processor 0 is one of the two hyperthreads of socket 0, core 0. The sibling of core 0 is core 12, which is the other hyperthread of socket 0, core 0. 

If we want to tune the system for better performance, we should know information about cache. We have to understand the sharing of cache among CPUs. To know this information, we can check files under `/sys/devices/system/cpu/cpu0/cache`:

```bash
test-server:[~]$ cd /sys/devices/system/cpu/cpu0/cache
test-server:/sys/devices/system/cpu/cpu0/cache$ ls
index0  index1  index2  index3  power  uevent   
```

the result shows there are four different caches used by cpu0 (L1d, L1i, L2, L3), but the number after *index* does not mean the cache layer, in order to know the cache layer and type, we can check the file `level` and `type` under each *index* folder:

```bash
test-server:/sys/devices/system/cpu/cpu0/cache$ cat index0/level index0/type
1
Data
test-server:/sys/devices/system/cpu/cpu0/cache$ cat index1/level index1/type
1
Instruction
test-server:/sys/devices/system/cpu/cpu0/cache$ cat index2/level index2/type
2
Unified
test-server:/sys/devices/system/cpu/cpu0/cache$ cat index3/level index3/type
3
Unified
```

now let's talk about how to inspect the sharing of caches among CPUs, for each cache represented by an *index* folder, there're two files which contain the CPU sharing information. The two files are `shared_cpu_list` and `shared_cpu_map`, they have the same information with different representations. For example, we can check which CPUs share the same L1d cache with CPU0 by:

```bash
test-server:/sys/devices/system/cpu/cpu0/cache$ cat index0/shared_cpu_map index0/shared_cpu_list
0000,00000000,00000000,00000000,00001001
0,12
```

and we know CPU12 shares L1d cache with CPU0, it does make sense because CPU0 and CPU12 are two hyperthreads of one CPU core. We can also check the sharing of other caches:

```bash
liuchang@fit137:/sys/devices/system/cpu/cpu0/cache$ cat index1/shared_cpu_map index1/shared_cpu_list
0000,00000000,00000000,00000000,00001001
0,12
liuchang@fit137:/sys/devices/system/cpu/cpu0/cache$ cat index2/shared_cpu_map index2/shared_cpu_list
0000,00000000,00000000,00000000,00001001
0,12
liuchang@fit137:/sys/devices/system/cpu/cpu0/cache$ cat index3/shared_cpu_map index3/shared_cpu_list
0000,00000000,00000000,00000000,00555555
0,2,4,6,8,10,12,14,16,18,20,22
```

and we know each CPU core (not each CPU/processor with a unique id which is actually a hyperthread) has it's own L1i, L1d and L2 cache, these caches are shared by two hyperthreads on that core. And each socket/physical CPU has it's own L3 cache which is shared by all 6 cores/12 hyperthreads on that socket (processor 0,2,4,6,8,10,12,14,16,18,20,22 have the same physical id).

