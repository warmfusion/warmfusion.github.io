---
layout: page
title:  "NFS vs iSCSI - My failed experiment"
tags: homelab 
share: true
comments: true
---

# NFS vs iSCSI in a Homelab: A Performance and Setup Experiment

This post documents my experience comparing **NFS** and **iSCSI** storage protocols in my homelab setup. 

During testing, I'd realised that my NAS was only ever able to provide 20MB/s throughput over the NFS, which significantly bottlenecked performance for my media management and data processing experiments.

This doc describes my (failed) attemps to find which protocol would offer better performance and manageability for media storage.

> TLDR: NFS ~25MB/s, isci ~8MB/s with CPU spikes on NAS. NFS sucks, but its _ok_.


## 🧪 Test Environment

**Client**:

* Debian Linux (bare-metal) HP-Gen8 MicroServer (16G RAM, Intel Xeon E3-1230 V2)
* Connected directly to NAS with a dedicated 1gbit/s path
    * Jumbo frames enabled (MTU 9000) and verified

**NAS**:

* Asustor AS6706T

  * 6x 3TB ST33000650NS CMR SeaGate enterprise NAS drives
  * 2x 1TB CT1000P3PSSD8 Crucial M.2 SSD cache drives
* Supports NFS and iSCSI
* LUN-backed storage volume (~4TB)


## 📁 NFS Setup and Performance

### Systemd Mount Unit

```ini
[Unit]
Description=NFS Mount to Media
After=network.target

[Mount]
What=192.168.2.15:/volume1/Media
Where=/mnt/nfs/media/volume1/Media
Type=nfs
Options=_netdev,auto

[Install]
WantedBy=multi-user.target
```

Viewing the output of `mount` shows that my Debian OS defaulted the NFS mount with the following options;

```bash
vers=4.2,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,_netdev
```

### FIO Test Command

> The Flexible I/O (FIO) tester is a versatile tool for measuring storage performance. It allows simulating various I/O patterns and workloads to evaluate how storage systems perform under different conditions. 

In this test, I used FIO to generate a mixed read/write workload with 128KB block sizes, which is somewhat representative of media file access patterns. 

The test runs for 60 seconds using asynchronous I/O (libaio) with a queue depth of 32 to measure sustainable throughput.

```bash
fio --name=nfs-test --directory=/mnt/nfs/media/volume1/Media \
    --size=1G --readwrite=rw --bs=128k --ioengine=libaio --iodepth=32 --runtime=60 --time_based
```

The results however, we're not promising, and aligned with my observed speeds...

```
Run status group 0 (all jobs):
   READ: bw=24.7MiB/s (25.9MB/s), 24.7MiB/s-24.7MiB/s (25.9MB/s-25.9MB/s), io=1480MiB (1552MB), run=60019-60019msec
   WRITE: bw=24.7MiB/s (25.9MB/s), 24.7MiB/s-24.7MiB/s (25.9MB/s-25.9MB/s), io=1482MiB (1554MB), run=60019-60019msec
```

So 25MB/s for both Read and Write... rubbish.

### Netcat Bandwidth Test

As those numbers were suspiciously similar, I wondered if there was a network bottleneck causing the issues, so while I have limited SSH access to the Asustor (running BusyBox), I used the handy netcat `nc` tool, along with pipeviewer `pv` to create a improvised network speed test...


The process is simple enough, you create a recieving socket that writes to /dev/null with `pv -l 12345 > /dev/null` and on the sending side you simply write a stack of data with `nc <host> 12345`. Pop a `pv` in the middle and get yourself a progress bar to show bandwidth usage.


```bash
# Client that recieves with throughput view
nc 192.168.2.15 12345 | pv > /dev/null

# Server that sends with throughput view
pv nfs-test.0.0 | nc -l -p 12345
```

These tests showed a typical result: ~117MB/s, which is pretty much spot-on 1gbit/s.


## 🔄 iSCSI Setup and Performance

So... the rabbit hole appears before me... lets see if block devices over iscsi will be performant.

Using the asustor, i setup a new LUN and Mount with a 4TB thinly provisioned disk, setup some CHAPS authentication and went to work connecting it to my server...

### Setup

```bash
# Install the iscsi stuff
sudo apt install open-iscsi

# And see what you can find announced from the NAS
iscsiadm -m discovery -t sendtargets -p 192.168.2.15
```

Once you've confirmed that the discovery works, you'll get a lun target such as `iqn.2011-08.com.asustor:as6706t-425067.media`

So onwards to setting up auth...

```bash 
# Setup auth
sudo iscsiadm -m node -T iqn.2011-08.com.asustor:as6706t-425067.media -p 192.168.2.15 --op=update -n node.session.auth.authmethod -v CHAP
sudo iscsiadm -m node -T iqn.2011-08.com.asustor:as6706t-425067.media -p 192.168.2.15 --op=update -n node.session.auth.username -v <username>
sudo iscsiadm -m node -T iqn.2011-08.com.asustor:as6706t-425067.media -p 192.168.2.15 --op=update -n node.session.auth.password -v <password>
# Save auth between boots
sudo iscsiadm -m node -T iqn.2011-08.com.asustor:as6706t-425067.media --op update -n node.startup -v automatic


# now login to the LUN and you should find yourself with a nice new drie
sudo iscsiadm -m node -T iqn.2011-08.com.asustor:as6706t-425067.media -p 192.168.2.15 --login
```

Once the login completes, you'll find that `lsblk` shows a new drive (in my case `/dev/sdf`) which you can go ahead and partition, format, mount etc as if it was a normal block device.


```bash
mkfs.ext4 /dev/sdf1
mkdir -p /mnt/media
# Quick mount to test - Note using the UUID ensures that if the drives reorder on boot, you don't mount the wrong things
mount UUID=$(blkid -s UUID -o value "/dev/sdf1") /mnt/media
```

### Systemd Mount Unit

Making the mount paths permanant...

```ini
[Unit]
Description=Mount iSCSI Media Volume
Requires=network-online.target
After=network-online.target

[Mount]
What=UUID=446c88c1-a51b-4ec4-9ada-9c18856d4b04
Where=/mnt/media
Type=ext4
Options=defaults,_netdev
TimeoutSec=30

[Install]
WantedBy=multi-user.target
```

### FIO Test Command

```bash
fio --name=iscsi-test --directory=/mnt/media \
    --size=1G --readwrite=rw --bs=128k --ioengine=libaio --iodepth=32 --runtime=60 --time_based
```

So I ran the FIO test again... and...

```
READ:  ~8.2MB/s
WRITE: ~8.2MB/s
```

> iSCSI performance was significantly lower, with notable CPU spikes observed on the NAS during tests.

What..?


## 📉 Observations

* iSCSI imposed heavy CPU load on the Asustor NAS, degrading throughput; Unclear why but some googling suggests thin-provisioning might be to blame
    * `media1_*` kernel threads pointed to LUN-related processing.
* NFS used less CPU and offered better throughput overall.


## ✅ Final Verdict

**NFS "wins"** for now... but i'm still pretty disapointed in the overall performance observed. 

More research is needed, as I bet theres something DUMB that i need to fix, but despite my efforts so far, i'm still stuck with a strangely slow NAS.

# Appendix

## 🪯 iSCSI Teardown Steps

### Unmount and Stop Mount

```bash
umount /mnt/media
systemctl stop mnt-media.mount
systemctl disable mnt-media.mount
```

### Logout and Delete Node

```bash
iscsiadm -m node -T iqn.2011-08.com.asustor:as6706t-425067.media -p 192.168.2.15 --logout
iscsiadm -m node -o delete -T iqn.2011-08.com.asustor:as6706t-425067.media -p 192.168.2.15
```

### Disable iSCSI Service (Optional)

```bash
systemctl disable iscsid
systemctl stop iscsid
```

### NAS GUI Cleanup

* Go to: **Storage Manager > iSCSI**
* Unmount and Delete the LUN and targets

---

