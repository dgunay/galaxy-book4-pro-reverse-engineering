## Galaxy Book4 Pro audio driver reverse engineering

I spent a few days trying to reverse engineer the Galaxy Book4 pro audio drivers to fix the speakers not producing sound on Linux. I was ultimately unsuccessful, but I figured the information I gathered in the process might help someone else later or eventually lead to someone smarter than me developing a quirk configuration patch or kernel patch to fix the driver on Linux. Maybe I missed something.

Caveats about my setup:
* I tried various parts of the process using both the `snd_hda_intel` and  `snd_sof_pci_intel_mtl` kernel modules.
* I did this all on EndeavourOS, kernel 6.8.9-zen1-2-zen.

## Getting Windows driver dumps/traces

I first tried the VFIO trace via qemu route. 

Here is jcs's hack for logging dma values: https://github.com/jcs/qemu/compare/e22f675b..jcs-hda-dma

I wasn't able to build his fork of qemu so I ended up
just manually patching the latest build of qemu. It worked with minimal adjustment (variable shadowing and some macro definitions didn't resolve correctly). I was still unable to get qemu to bind to the sound card properly and get passthrough. 

I personally found the RtHDDump.exe path much, much easier than the qemu path so I'd recommend just keeping your Windows partition around and doing that. My own output is included here in this repo, but if you want I've also included the executable in a zip file. Boot into windows, run it, and dump the file.

## Observations 

I didn't find any differences in the pin configurations.

I converted the HDA verbs from the linux driver dump into the same format as the Windows RtHDDump.txt format.

If you run 

```console
diff ./windows-converted-hda-verbs ./linux-converted-hda-verbs
```

you'll see:

```diff
< Index 0x10 0x0E21
---
> Index 0x10 0x0C21
```

That appears to be the sole difference, but maybe there's more to it. My understanding of how this all works is very poor. I wasn't able to get any results from poking around `hda-analyzer`, `hdajacksensetest`, and `hdajackretask`. By the way I've included a version of `hda-analyzer` that runs on Python 3 in the `alsa` folder. `sudo python ./alsa/hda-analyzer/hda-analyzer.py` should work.

## Other resources

I primarily followed https://asus-linux.org/blog/sound-2021-01-11/, as that's what got me the furthest.
