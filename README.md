# Linux Boons
A collection of solutions to linux issues I have faced and found solutions to that may or may not be niche, this list will enevitably grow. I hope these may be of use to at least some lost soul.

# Issue: xwayland apps on KDE Plasma are pixelated/blurry/wrong-resolution
Also shows as: xrandr for xwayland shows incorrect resolutions of active monitors ; xrandr and xwayland "xrandr failure xrandr returned error code 1 Error of failed request: Badmatch (invalid parameter attributes) etc

![image](./resources/Xrandr%20Error.png)

**Solution: Remove any monitor scaling set in the KDE display app, KDE may erroneously apply a scaling to some monitors**

![image](./resources/Scaling.png)

*Scaling must be 100% (normal) for all monitors*

# Issue: System and i/o freezes with slow SSD speeds with dm-crypt drives when under heavy loads
Also presents as: Absurdly slow write/read speeds for an ssd in the ~2MB/s range while using dm-crypt/luks not connected to crypto speed; plumetting write speeds after linux disk cache is full; **System Freeze during heavy ssd i/o using dm-crypt**; Heavy i/o on one dm-crypt device freezes i/o system wide

This issue is a rather peculiar one as it changes how its presented depending on the circumstance, but it boils down to some disconnect between ancient code involving synchronous/async caching in dm-crypt and linux that is covered here [here](https://blog.cloudflare.com/speeding-up-linux-disk-encryption/) and *some* disk controllers (I specifically experienced it with a 'Crucial P2 2TB 3D NAND NVME' and no others). I suggest a read, however it is not entirely neccesary as I'll cover what ive found to remedy it, though it will not entirely solve the issue, merely subdue it to a tolerable degree. I suspect its an issue with specific SSD controllers as adding a heat sink increases said abysmal performance 2x.

**Solution: Disable synchronous disk writes and/or use linux-zen kernel**

First and foremost I'de suggest just trying the zen kernel as it seems to for the most part fix this and other niche issues, but not always an option to change kernels so I'll cover multiple options. Some, one, or all of these options may be neccesary to improve performance to an acceptable level, but I may add I was never able to fully achieve above ~10MB/s writes after filling the disk cache but the i/o and system freezes were nullified. I enventually had to swap said drive and never managed to get it to behave.

1. Use the linux-zen kernel
2. When opening the drive using cryptsetup pass the following flags (`--perf_no_read_workqueue --perf_no_write_workqueue`) to disable the cache (linux-zen *should* make these flags the default, but I've had inconsistent results relying on zen alone to pass them.)
3. Change [disk scheduler](https://wiki.archlinux.org/title/Improving_performance#Input/output_schedulers). The results of this may vary greatly so I suggest following the guide and testing each, but in my case I had the best luck just disabling it entirely, strange as it may seem.
4. Improving thermals: I was able to increase performance 2 fold by adding a heatsink to the SSD in question, despite never facing issues cooling other drives in worse conditions, you may face some luck here as the speed increase was instantly noticable.

In particular if you're just concerned about i/o and system freezes 1 & 2 are the most impactful in remedying those.

Should these not fully solve it, you may delve deeper as [cloudflare](https://blog.cloudflare.com/speeding-up-linux-disk-encryption/) has more additional patches related to this issue that never fully made it in you may have luck with. 
It may also be of interest if you're interested into why this is an issue in the first place, as it is not because cryptography is computationally expensive (In this case).
