Dummy NVMFS to inject arbitrary latency
=================================
This is a minimalistic dummy filesystem based on tmpfs to emulate NVRAM with arbitrary latency.
It is used in some performance tests that evalute systems performance on various NVRAM latency
settings.

We do *NOT* mean this is the best way to emulate NVRAM. Read the pros/cons below.

**Pros**

* Handy. Most existing programs that access filesystems would work without any modification.
* Portable. You don't need any special hardware or software.
* Arbitrary latency. You can emulate whatever latency you want to emulate.
* NUMA-aware. Just like tmpfs, it respects libnuma's affinity setting.

**Cons**

* Overhead: It incurs filesystem's overhead. If you are reading/writing just one byte, filesystem
API call would be the majority of the cost. If you are reading/writing kb, doesn't matter.
* Capacity: It uses DRAM after all. The capacity is limited by your DRAM.
* Durability: Again, it's DRAM after all. You can't test the non-volatility after system crash.


Disclaimer (READ BEFORE USE!)
-------
**You need to compile, configure, deploy a linux kernel** to use this module.
It can **cause arbitrary damage to your system** if done incorrectly.
**Never, EVER, try this at home** unless you take the whole system backup beforehand and
you have a reasonable amount of experiences on linux hacking.
We will not take any reposibility or whatsoever on the outcome.


Steps to Compile/Deploy
-------
I assume you have done linux kernel compilation yourself before.
Otherwise, please do not try this by yourself. PLEASE.

* Download upstream linux kernel code from https://www.kernel.org/.
**We recommend [3.16.7](https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.16.7.tar.gz)**.
Due to API changes in linux, we observed a compilation error in 3.18.
Say you inflated it to /tmp/linux-x.x.x/
* Copy this folder as /tmp/linux-x.x.x/fs/nvmfs
* Add the following line to /tmp/linux-x.x.x/fs/Makefile

    obj-y += nvmfs/

* Compile and deploy the kernel. Make sure you enable NUMA and
other esssential things. We also put sample_kernel.config in this folder
*as a reference*. You should figure out the right configuration
for your machine yourself.
* After building and installing the new kernel you should be able to
mount/umount nvmfs filesystems.

Parameters
-------
You can specify the following parameters when you mount the dummy fs.

* rd_delay_ns_fixed
* wr_delay_ns_fixed
* rd_delay_ns_per_kb
* wr_delay_ns_per_kb
* cpu_freq_mhz

For example,

    sudo mount -t nvmfs -o rd_delay_ns_fixed=500,wr_delay_ns_fixed=500,rd_delay_ns_per_kb=0,wr_delay_ns_per_kb=0,cpu_freq_mhz=2800,size=1000000m nvmfs /testnvm


Hopefully the parameter names are self-explanatory.
Just one clarification, the delay is injected via x86 RDTSC, so it needs to know
the CPU cycle frequency. Unfortunately it is not easy to measure.

Today's CPU has a dynamic clock-speed management and
an advanced config (so-called *turbo-mode*).
You must specify the clock speed accounting for them.
What we usually do is as follows.

    # Linux-side config. Remember, this is NOT enough in some environments.
    sudo su
    for CPUFREQ in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do [ -f $CPUFREQ ] || continue; echo -n performance > $CPUFREQ; done
    # Also configure it in BIOS. Check with the manufacture how to do it.
    # Then, reboot.
    # Or, run something to make your CPU really busy.
    more /proc/cpuinfo # Finally, see the clock speed.

Tips
-------
* kexec is great. You can switch kernel in a minute.
* Once your experiments are done, I strongly recommend to rollback to the original
kernel installed by your distro. It sometimes causes a catastrophic issue with
the distro's package manager (eg yum). It has once razed my grub entries, leaving the machine
completely unbootable. Again, **DO NOT TRY THIS AT HOME**.
* Make sure you keep your kernel.config. You will soon forget which options you picked.

Acknowledgement
-------
This work is done by the filesystem hacker, Haris Volos.
I just ate lunch while he made it in an hour.

License
-------
GPLv2. This is a modified version of tmpfs, which is linux's code.
