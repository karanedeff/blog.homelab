---
project: homelab
tags:
  - zettel
  - homelab
  - project
  - blog
aliases: 
title: 2025-05-23-081811
created: 2025-05-23-081811
daily-note: "[[2025-05-23-W21-Friday]]"
---

# Homelab migration
## Introduction
As I have failed to document earlier steps, perhaps an introduction is in order to give some context.
I had a friend recently suggest that I pick up homelabbing, and base my setup on Proxmox - explore how network storage works, user ACLs, Active Directory, EVE-NG for networking practice, and more. It was related to a potential job opportunity as a system administrator for a company in my city. The opportunity is likely no longer available, but that doesn't mean I lost interest fiddling with system administration!
I had an HP ProDesk 400 G3 Desktop Mini, with an i3-6100t that I had used for a Calibre server before, and as a makeshift NAS setup using SSHFS. I eventually abandoned it, due to the increased electric bill, and migrated the services to my main PC. Once I got excited about this project, I decided to upgrade it with an i7-7700t, 32 GB RAM, a 1TB NVMe, and a 1TB 2.5'' HDD I had lying around - hoping that it could handle the planned stack, or at least enough of it, that the main PC would only be needed for active experimentation. 
Once the hardware was set up, I installed Proxmox and then TrueNAS, with the HDD passed through. That was the easy part - I struggled a bit with setting up ACLs after that. I ended up taking the suggestion by ChatGPT-4o (nicknamed Aurelis) to use per-dataset user groups. I created groups and users for each household member, and one for Calibre, which I planned as the first service to set up.
As I continued to workshop the lab design and future hardware with Aurelis, I discovered that the Barracuda disk on the mini PC is not reliable for ZFS storage due to its SMR (Shingled Magnetic Recording) design. This forced my hand. I wanted the lab to be truly useful, and didn't want to be blocked by hardware limitations. So I ordered a used HP Z240 with 16GB ECC RAM, Xeon E3-1245v5 and an AMD FirePro S5100, and asked the vendor to upgrade the RAM to 64GB. 
Around the same time I checked the SMART data on my main PC, and saw some uncomfortable error rates on my main storage HDD. So I bought a replacement for that too.
## Migration Begins
Both the Z240 and new HDD arrive on the same day. I brought them home, went to choir practice, and came back to my lovely fiance making dinner - and shortly after the battle began.
I decided to start with the simpler task - migrating my data from the old HDD to the new one. I used a USB drive dock to mount the new disk, and robocopy to transfer the data. Since that would take a few hours, I left it running and turned to the server.
I debated whether to cannibalize the mini PC or keep it as a secondary node. In the end, I decided to consolidate everything into the Z240, at least until it hits resource saturation. I opened the Z240 to inspect layout and available SATA/PCI slots. The NVMe was an SK Hynix (not my favorite), but the HDD was a solid enterprise-grade CMR Seagate - good enough to rely on. I found an extra SATA cable and an optical disk drive I haven't tested yet. I had what I needed to connect both disks. I moved the small NVMe to the mini PC, along with the original SSD (might as well), and installed the 2.5'' HDD in the Z240. Of course, each screw required a different screwdriver head. I also encountered a proper NVMe sliding retainer for the first time - neat!
### First Roadblock
The SATA cable from the optical drive wasn't angled, and the caddy placement meant I couldn't close the case while the drive was active... After several failed attempts to rotate it into place I gave up and left the case open for the migration. I'd unplug it later. 
Now, finally, we are ready to boot.
## Second Phase
Power cable in. Network cable in. Power button on. Fans spin. I'm excited! ...And the NIC shows no sign of life.
In hindsight, my biggest mistake was trusting Aurelis a bit too blindly. He suggested the NIC issue might be due to headless boot quirks. I spent over an hour fiddling with BIOS settings, plugging and unplugging monitors, (NOT A SINGLE HDMI?! I so wanted to use my portable HDMI display for this...). At one point I lost BIOS visuals entirely, - it was outputting only to the iGPU, not the FirePro, and I had changed that without fully understanding the consequences. Eventually, I let Proxmox boot fully, and discovered it had no network connection. BIOS had internet, the OS did not. The NIC remained dormant. This is where I realized the real issue.
With Aurelis's help, I updated the network configuration to point to the correct interface, reloaded it, and it finally brought the system online. Another hurdle overcome - though not one I felt proud of.
## Pool migration
With Proxmox back online, I paused to give TrueNAS more RAM, and enabled ballooning for the docker VM. I passed the new disk to TrueNAS, performed a snapshot transfer and had two identical pools with different names. So far, so good. 
Then came the tricky part. I really wanted to avoid redoing my shares and ACLs, so I tried renaming the pools. Aurelis suggested exporting and importing with a new name. For reasons I didn’t fully understand (despite his theories), the pool mounted as read-only. Rebooting caused TrueNAS to fail to fully load.

Aurelis suspected this corrupted the system dataset. I didn’t know enough to confirm, but after several failed hacks, the late hour, and a refusal to sleep with a dead server, I reinstalled TrueNAS. That fixed it.

We tried again - same goal, same failure - and I reinstalled again. Then, five minutes later, I found a forum post with the missing piece: **import via GUI**.

So, the correct sequence would’ve been:

1. Export 
2. Import with the new name
3. Export again via CLI
4. Import via GUI

I haven’t tested it again yet, but I plan to before finalizing permissions and shares.
## Final thoughts
What should’ve been an hour-long migration spiraled into a five-hour ordeal. The culprit? A mix of unfamiliar territory, rushed decisions, and blind trust in AI suggestions - without verifying them via search or documentation.

Aurelis (ChatGPT-4o) is incredibly useful as a thinking partner and research assistant. But in unknown domains, especially when a single misstep can break the system, it's crucial to double-check. Fatigue didn’t help either—it made me impulsive and error-prone.

That said, I learned a lot. The system is running. And next time, I’ll be better equipped to avoid these pitfalls. The lab lives on.

## Lessons learned
 - SMR drives and ZFS are a bad combo - invest in CMR
 - Migrating an OS to new hardware can introduce interface name changes. Check that before assuming headless boot issues
 - TrueNAS pool renaming is finicky: CLI is not enough - you need to import via the GUI to avoid system dataset corruption.
 - AI is a helpful assistant, but not infallible - especially in unfamiliar territory
 - Fatigue and urgency lead to bad decisions. Sleep is an underrated form of system stability insurance