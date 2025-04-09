# QTI-mem-opt

Memory management optimization for Android platforms.  

## Feature

- Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms
- Customizable list of protected APPs, preventing them from being killed by Android in-kernel lowmemorykiller
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- Reduce jitters under high memory pressure, adjust the trigger threshold and execution interval of lowmemorykiller, and keep the file page cache at a high level
- Reduce stucks under high memory pressure, reduce the probability of direct memory allocation via higher extra_free_kbytes
- Allow the system to consider having more cache than default via watermark_low
- Disable adaptive lowmemorykiller
- Prohibit kernel memory recycling threads running on the prime core, avoid congesting the main thread that is interacting and reduce energy consumption
- Avoid swapping memory pages which are hard to compress to ZRAM, make the compression rate close to the ideal value of 2.8x
- Make Android run better through ART optimization from Master ART and some custom ROMs focused on optimization
- Customizable ZRAM size and compression algorithm(needs kernel support), ranging from 0G to 6G
- SELinux can still be enabled

## How to use?
- Install the module and reboot. Open the file /sdcard/Android/panel_qti_mem.txt to modify the ZRAM size and the desired compression algorithm. The changes will take effect after reboot.

- Open the file /sdcard/Android/panel_adjshield.txt to add the packages of the apps you want to keep in the background. The changes will take effect after reboot.

- The default ZRAM size values ​​are:

- 1-2GB RAM: Enables 300-900mb of ZRAM with swappiness of 100.

- 3-4GB RAM: Enables 1gb-1.4gb of ZRAM with swappiness of 100.

- 6-8GB-12gb RAM: Enables 2.1-2.8gb of ZRAM with swappiness of 40-20.

- ZSWAP is not supported at the moment

- Userspace LMK is supported, it is the current LMK, abandoning the optimizations with the old LMK.

## FAQ

### Usage Answers

Q: What is this? Is it a one-click full optimization?
A: This is a Magisk module that improves Android cache process management, avoids clearing background cache processes too quickly and improves fluency in low memory conditions. It does not include other parts such as CPU scheduling optimization.

Q: Can my device use this?
A: This module is applicable to Android 32/64-bit platforms with Android version >= 6.0, not limited to Qualcomm platforms.

Q: If ZRAM is not enabled, will this module be useless?
A: ZRAM control is only a small part of the function of this module. Using this module without ZRAM can also improve fluency in low memory conditions. The display of `unsupported` in the ZRAM item of the panel file only means that the kernel does not support ZRAM. Other parameter modifications and cache control are still effective.

Q: My device has 12GB or 16GB of physical memory. Do I still need this module?
A: On some devices, due to the strict limit on the number of cached processes on the Qualcomm platform, the background cache will be cleared even if there is a lot of available memory. This module avoids clearing the background cache process too quickly so that the large memory can be fully utilized.

Q: Why is ZRAM not enabled after the ZRAM size is set in the configuration file?
A: This module cannot be added if the kernel does not have the ZRAM function. Most official kernels support ZRAM, and third-party kernels do not support ZRAM more often.

Q: Is there a problem with SWAP usage of 100%?
A: No problem, it is better to say that this is the expected result. ZRAM is an implementation of swap partitions. It compresses and stores infrequently used memory pages. The compression rate is generally 2.8x, which means that 2.8G of ZRAM occupies 1GB of space in physical memory, which is equivalent to an additional 1.8G of memory. The impact of ZRAM on performance and power consumption depends on the ZRAM size you choose to enable. The larger the setting, the higher the probability that the CPU needs to decompress and read, and the more processes can be cached in the background at the same time. The system fluency has no direct relationship with the SWAP usage rate, but depends on the difficulty of memory recovery and the page cache hit rate.

Q: Why does the background still drop?
A: Physical memory resources are limited and it is impossible to meet the unlimited background cache requirements. Some manufacturers may have additional background cache management, such as using LSTM prediction to select and clean up the APP that is least likely to be used next. If you need to protect some APPs from being recycled by kernel-mode LMK, you can add the package name to be protected in the `AdjShield` configuration file. It is not recommended to add too many APPs to avoid difficulties in memory recovery.

Q: Why is the power consumption increased?
A: The cache process itself does not increase power consumption, and the power consumption increased by more page swaps is very limited. It should be noted that while keeping more background APPs alive, not all of these APPs are in cache dormancy, and there may be many services running in the background that consume power.

### Technical Answer

Q: What is the use of pinning system commonly used files in the file page cache?
A: In Google's [Android performance tuning document](https://source.android.com/devices/tech/debug/jank_jitter#page-cache), it is mentioned that the jitter of the page cache under low memory conditions is the main cause of long lags. In practice, it manifests as a pause of more than 100ms when returning to the desktop, which is particularly obvious in the gesture animation of returning to the desktop. Generally speaking, the Android framework's `PinnerService` has fixed commonly used files in memory, but devices on some platforms, such as OnePlus 7Pro, do not have this service, or the coverage of the fixed memory is not large enough, resulting in key page cache jitters. This module fixes most of the system's commonly used files in the file page cache to make up for the imperfections of the existing settings.

Q: How to prevent specific apps from being cleared by Android kernel-mode LMK?
A: Even if the LMK trigger threshold is increased and the SWAP space is increased, some key apps may still not be able to survive in the background all the time, not to mention wanting to keep large games and commonly used chat software alive at the same time on devices with less than 4GB of physical memory. In the previous solution, the Xposed framework is needed to keep the specified APP alive, but some people don't like the Xposed framework (such as me). The Android system framework itself or notifies the user-mode LMKD to call the `procfs` interface to change the APP's `oom_score_adj`. The kernel-mode LMK intervenes when the page cache is insufficient to terminate the process with the highest `oom_score_adj`. The `AdjShield` of this module regularly traverses `procfs` to match the APP package name that needs to be protected, intercepts the write operation to its `oom_score_adj`, and ensures that the APP that needs to be protected will not be the first to be terminated by the kernel-mode LMK (including simpleLMK). The `oom_score_adj` of the protected APP is fixed at 0. The interval of regular traversal is set to 2 minutes, and the time consumption of each traversal is optimized to be controlled within 40ms (Cortex-A55@0.8G), which will hardly cause additional burden on battery life and performance.

Q: What is the relationship between ZRAM and swap?
A: ZRAM is a swap partition implementation. When the kernel reclaims memory, inactive anonymous memory pages are swapped into a block device, which is called swap. This block device can be an independent swap partition, a swapfile, or ZRAM. ZRAM compresses the swapped pages and puts them into memory, so it has several orders of magnitude lower read and write latency than traditional swap methods, and has better performance.

Q: Why not use swapfile?
A: The read and write latency of swapfile stored in external storage such as flash or disk is several orders of magnitude higher than that of ZRAM, which will significantly reduce the fluency, so it is not used.

Q. Why are you using ART optimizations?
A. ART itself is the way for applications to run on Android. On current Androids, when ART is optimized, it consumes less and less memory. So, optimizing ART on stock ROMs can improve memory management, such as system_server, for example.

Q: Which one is better, this or SimpleLMK?
A: It is inappropriate to compare Magisk modules with kernel modules. It is more appropriate to compare SimpleLMK with LMK. SimpleLMK is triggered on direct memory allocation, and LMK is triggered when the file page cache is below the threshold after kswapd reclaim ends. SimpleLMK is triggered later, and its advantage is that it can use all memory to store active anonymous pages and file page cache as much as possible. The disadvantage is that the file page cache may have extremely low values, causing relatively long pauses. LMK is triggered earlier. Its advantage is that it actively maintains the file page cache level and is not likely to cause long pauses. Its disadvantage is that it is easily affected by cache level fluctuations, which may lead to the mistaken clearing of background cache processes. This module adjusts the execution cost of LMK and alleviates the problem that LMK is easily affected by cache level fluctuations.

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
