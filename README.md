# qti-mem-opt

Memory management optimization for Android platforms. Focused on making Android itself more intelligent, adaptable and responsive to user demand. Respecting its limits and, above all, respecting its user.

## Features
- Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms. And most importantly: follow the ideology that free RAM is wasted RAM, and follow the ideology that overused RAM is waste. Be the middle ground and be efficient, meet memory demand, allow for reserves, avoid underutilization and improve overall system memory utilization, avoid waste, overuse and make Android smart enough to adapt to almost any memory situation, if not all
- Customizable list of protected APPs, preventing them from being killed by Android in-userspace lowmemorykiller
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- Avoid stutters and jitters in high-load situations. Avoid micro-lags that occur when the device is using a lot of memory, maintain responsiveness even under pressure
- Reduce jitters under high memory pressure, adjust the trigger threshold and the kill timeout of lowmemorykiller daemon, and keep the file page cache at a high level
- Reduce stucks under high memory pressure, reduce the probability of direct memory allocation via higher extra_free_kbytes
- Disable adaptive lowmemorykiller daemon
- Prohibit kernel memory recycling threads running on the prime core, avoid congesting the main thread that is interacting and reduce energy consumption
- Reduce swapping costs and unnecessary situations as much as possible. If swapping is occurring, be as efficient as possible and allow above-average throughput for swapping
- Avoid swapping memory pages which are hard to compress to ZRAM, make the compression rate close to the ideal value of 2.8x
- Use the UFFD garbage collector, allowing to reduce the chances of page faults and allowing ZRAM to compress better to an acceptable level, without increasing the cost of swapping
- ART optimizations of the Android runtime. In order to design better app execution time, and in turn: a few megabytes of less consumption
- Introduce Hybrid Swap! A memory management technique that combines swapfile and ZRAM with Qualcomm's PPR. Allowing to increase effective memory by up to 17% with just 512mb of swapfile and with swapping costs reduced by up to 27%, and even better: it prevents storage from suffering, allowing to keep its useful life up to date! However, it is only for phones with snapdragon processors. Credits to: unintellectual-hypothesis @ github
- Prevent apps from being swapped out or killed too quickly. Allowing browsers like Brave to function even in memory-limited situations
- Customizable ZRAM size and compression algorithm(needs kernel support), goes from 0GB to 8GB
- Customizable swapfile size. Going from 0GB to 3GB. But you need hybrid swap active. Where in non-snapdragon phones, the hybrid swap will be just swapfile + zram, while in snapdragon it will be swapfile + zram + ppr
- SELinux can still be enabled

## Requirement

- ARM/ARM64
- Magisk or KSU
- Android 10-15
- 2GB or more of memory, because the module adapts to current values, where the current Android system needs AT LEAST 2GB of RAM to work. So values ​​smaller than 2GB will not be able to use the module for efficiency reasons

## Installation
- Do not download the repository version, because I am at the beginning of my career as a module creator, avoid using the repository version (that is, one that is not in the releases tab) because I do not know how to update all the files for the repository (I am new to github, sorry)
- Magisk or KSU, preferably the most updated version
- Install this module and reboot, open `/sdcard/Android/panel_memcfg.txt` to modify the parameters, such as disabling ZRAM to put a swapfile in its place, activating hybrid swap + swapfile, changing the size and compression algorithm of ZRAM, and this will take effect after reboot
- Open `/sdcard/Android/panel_adjshield.txt` and add the package name of the APP that needs to be kept in the background. It will take effect after reboot.
- ZRAM values ​​are based on AOSP ROMs. This means that in turn, the RAM values ​​will be between half of the fixed RAM of your device, such as:
  - 2GB of RAM gets 1GB of ZRAM by default
  - 3GB of RAM gets 1.5GB of ZRAM by default
  - 4GB of RAM gets 2GB of ZRAM by default
  - 6GB of RAM gets 3GB of ZRAM by default
  - 8GB of RAM gets 4GB of ZRAM by default
  - 12GB or more gets 6GB of ZRAM by default
  - Swapfile and Hybrid Swap will come disabled/set to 0, requiring user activation for them to work
  - Processors other than Snapdragon such as MediaTek can use hybrid swap, but in its most "basic" form. This means that the technique will not be 100% effective due to the lack of Qualcomm's PPR, providing only the basic swapfile
- ZSWAP is not supported currently, after all, we have hybrid swap that does almost the same thing, but if many users want it, I can add ZSWAP support, but only if there are users who use this scheme
- The LMK in userspace is the LMK used for optimizations, old LMK is no longer supported by the module
- In the future I plan to add support for LMKD with PSI. For now I only support the default minfree (which is what I have in my kernel).

## FAQ

### Use Answers

Q: What is this? What is the purpose of this fork compared to Matt Yang's original module?
A: This module for magisk and KSU is a re-adaptation of Matt Yang's module, with the focus of not only improving Android's cache management, but also improving the overall memory management of the device. Overall, its focus is to improve Android's memory management intelligence, and in turn: Allow the device to perform better in most, if not all, memory usage situations (light multitasking, moderate multitasking, gaming, etc.), while maintaining consistency even in low memory situations. It does not include other parts, such as CPU scheduling optimization.

Q: Will my device work with this?
A: This module is applicable to 32/64-bit Android platforms with Android 10-15, not limited to Qualcomm platforms.

Q: If ZRAM is not enabled, will this module be useless?
A: ZRAM control is only a small part of the module's functionality. Using this module without ZRAM enabled can also improve performance in low memory situations. The display of `unsupported` in the ZRAM item of the panel file only means that the kernel does not support ZRAM. Other parameter modifications and cache control are still effective.

Q: My device has 12GB or 16GB of physical memory. Do I still need this module?
A: On some devices, due to the strict limit on the number of cache processes, the background cache may be cleared even if there is a lot of memory available. This module prevents the background cache process from clearing too quickly, so that the large memory can be fully utilized.

Q: Why is ZRAM not enabled after setting the ZRAM size in the configuration file?
A: If the kernel does not have the ZRAM function, this module cannot be added. Most official kernels support ZRAM, but third-party kernels usually do not support ZRAM.

Q: Is it acceptable to have 100% SWAP usage?
A: No problem, this is the expected result. ZRAM is an implementation of swap partitioning. It compresses and stores infrequently used memory pages. The compression ratio is typically 2.8x, which means that 2.8G of ZRAM takes up 1GB of physical memory, which is equivalent to an additional 1.8G of memory. The impact of ZRAM on performance and power consumption depends on the size of ZRAM you choose to enable. The higher the setting, the more likely the CPU will need to decompress and read, and the more processes can be cached in the background at the same time. System fluency is not directly related to SWAP usage, but depends on the difficulty of memory reclaiming and the page cache hit rate.

Q: Why does the background still crash?
A: Physical memory resources are limited and cannot meet the unlimited background cache requirements. Some vendors may have additional background cache management, such as using LSTM prediction to select and purge applications that are least likely to be used next. If you need to protect some applications from being recycled by kernel-mode LMK, you can add the package names that need to be protected in the `AdjShield` configuration file. It is not recommended to add too many applications to avoid difficulties in memory recycling.

Q: Does the module reduce or increase power consumption?
A: It depends. In general, I improved Matt Yang's engineering and made swapping a "fallback" in general. In other words, the system avoids using swapping frequently in light use, which saves energy. But when necessary, it uses swapping with a higher throughput than average, but respects Android's limits and scales usage as the device uses memory. In general, consumption may increase or decrease depending on your use of apps. Swapping scales with usage, being more intelligent than Android itself, even Android 15.

### Technical Answer

Q: What is the use of pinning commonly used files by the system to the file page cache?
A: In Google's [Android Performance Tuning document](https://source.android.com/devices/tech/debug/jank_jitter#page-cache), it is mentioned that page cache jitter under low memory conditions is the main cause of long delays. In practice, this manifests itself as a pause of more than 100ms when returning to the desktop, which is particularly evident in the animation of the return to the desktop gesture. Generally speaking, the Android framework's `PinnerService` pins commonly used files to memory, but devices on some platforms, such as the OnePlus 7Pro, do not have this service, or the pinned memory footprint is not large enough, resulting in significant page cache jitters. This module pins most of the commonly used files by the system to the file page cache to compensate for the imperfections of the existing configuration.

Q: How to prevent specific applications from being cleaned up by the Android userspace-mode LMK?
A: Even if the LMK activation threshold is increased and the SWAP space is increased, some important applications may not be able to survive in the background all the time, not to mention the need to keep large games and commonly used chat software active at the same time on devices with less than 4 GB of physical memory. In the previous solution, the Xposed framework was required to keep the specified application active, but some people don't like the Xposed framework (like me). The Android system framework itself notifies the user-mode LMKD to call the `procfs` interface and change the `oom_score_adj` of the application. The kernel-mode LMK intervenes when the page cache is insufficient to kill the process with the highest `oom_score_adj`. The `AdjShield` of this module regularly traverses `procfs` to match the package name of the APP that needs to be protected, intercepts the write operation to its `oom_score_adj`, and ensures that the APP that needs to be protected is not the first one to be terminated by the kernel-mode LMK (including simpleLMK). The `oom_score_adj` of the protected APP is fixed to 0. The regular traversal interval is set to 2 minutes, and the time consumption of each traversal is optimized to be controlled within 40ms (Cortex-A55@0.8G), which is unlikely to cause additional overhead on battery life and performance.

Q: What is the relationship between ZRAM and swap?
A: ZRAM is an implementation of swap partition. When the kernel reclaims memory, inactive anonymous memory pages are swapped to a block device, called swap. This block device can be an independent swap partition, a swap file, or ZRAM. ZRAM compresses swapped pages and places them in memory, which makes it several orders of magnitude lower in read and write latency than traditional swapping methods, in addition to offering better performance.

Q: Why does the fork use swapfile and hybrid swap in general, unlike the original module?
A: Even with the disadvantages of swapfile by itself, it is a good fit for having a "fallback" memory. In this case, in situations where the device needs memory, it can allocate less used data in the swapfile. Due to this, hybrid swap is used, even with the limitation of working only on Qualcomm devices because only them have PPR (per-process reclaim), hybrid swap allows the user to reduce swapping costs and storage degradation by up to 28%, which is even higher than ZSWAP itself, which reduces it by only 26%. This generally allows users to use the swapfile as fallback memory, and makes swapping generally less expensive, resulting in a higher throughput device overall. If you want the source, here it is: https://ieeexplore.ieee.org/document/8478216

Q: Why is the swapfile only in /data? And not in /data/swap?
A: Ironically, placing the swapfile only in the /data directory reduces the latency of accessing the swapfile by up to 50%, and also allows compatibility on devices with more restrictive SElinux. Overall, it improves the latency of the swapfile and prevents the storage from being used more than it should be; overall, it's all gains and no losses.

Q: Which is better, LMKD minfree, LMKD PSI, or SimpleLMK?
A: It is inappropriate to compare Magisk modules to kernel modules. It is more appropriate to compare SimpleLMK to LMKD minfree or PSI. SimpleLMK is triggered on direct memory allocation, and LMKD in minfree format is triggered when the file page cache is below the threshold after kswapd has finished reclaiming, while LMKD in PSI format is much more efficient and better at detecting pressure, because it uses VMPRESSURE better than minfree itself when using CPU usage as a basis (based on ZRAM, seeing if it is compressing too much for the CPU), amount of user apps and I/O, allowing LMKD to make decisions based on respecting the user and in their usage, like in games. Compared to the three formats, I would put it this way: SimpleLMK > LMKD PSI > LMKD Minfree in terms of general use without needing modifications, of course, a LMKD Minfree may end up being better, just like the LMKD PSI, but all of this is just a matter of optimization.

Q: What is UFFD and why has it reduced ZRAM usage?
A: UFFD is a garbage collector that has been around since Linux 4.4. Its function is to ensure that unused pages are cleaned up. It is not used on all devices because it is complex and can generate additional overhead. However, on current Android devices, the UFFD userspace function is added, allowing for a significant reduction in overhead. The reason ZRAM is used less because of this is because these unused pages would be sent to ZRAM. This is generally more beneficial because it prevents kswapd from working too hard, allowing swapping to be more effective by having "cleaner" pages to use.

Q: Why optimize Android Runtime (ART)?
A: Optimizing the Android runtime allows the system to execute even LMKD better, making it even capable of executing other services such as GC, cache control, etc. Allowing memory management in general to also become more accurate by making it more native to the system.

Q: Why don't you use ZRAM Deduplication?
A: Deduplication by itself is very CPU-intensive, draining a lot of battery and affecting performance in CPU-hungry games. Because of this, I will EITHER leave ZRAM deduplication as an optional (i.e. user-enabled) feature, OR not use it if I can get more effective results than relying on deduplication, it just depends on my focus.

## Credit

@Doug Hoyte  
@topjohnwu  
@卖火炬的小菇凉--改进在红米K20pro上的zram兼容性  
@钉宫--模块配置文件放到更容易找到的位置  
@予你长情i--发现与蝰蛇杜比音效共存版magisk模块冲突导致panel读写错误  
@choksta --协助诊断v4版FSCC固定过多文件到内存  
@yishisanren --协助诊断v5版FSCC在三星平台固定过多文件到内存  
@方块白菜 --协助调试在联发科X10平台ZRAM相关功能  
@Simple9 --协助诊断在Magisk低于19.0的不兼容问题  
@〇MH1031 --协助诊断位于/system/bin二进制工具集的不兼容问题  
