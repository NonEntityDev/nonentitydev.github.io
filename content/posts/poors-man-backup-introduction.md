+++
date = '2025-08-28T10:31:00+01:00'
title = 'My poor man backup and NAS: Part 1 - Breaking the inertia'
publishDate = '2025-08-28'
categories = ['Automation']
tags = ['backup', 'linux', 'bash']
summary = 'Tired of relying on overpriced, privacy-invasive cloud services, I finally decided to build my own backup system — something low-cost, reliable, and fun to tinker with. In this post series, I’m documenting my journey to set up a 3-2-1 backup routine using what I already have: a Raspberry Pi, some spare drives, and a bit of scripting. The goal? Keep my data safe without breaking the bank or overcomplicating things.'
+++

## TL;DR
- I need a very low-cost but reliable backup routine, which I’ve been neglecting for too long.  
- I want to follow the **3-2-1 backup rule**: 3 copies of my data, 2 different storage media, 1 off-site storage.  
- I calculated my storage requirements based on real usage.  
- My starting setup: 1 Raspberry Pi 4B (2GB RAM), 1 × 256GB SSD, 1 × 1TB USB HDD, and AWS S3 Glacier Deep Archive.  

## Introduction

I have been using cloud storage systems as my main way to sync and **backup** data for the past 10 years (at least). Yes, I know, cloud storage like OneDrive, Goole Drive, Dropbox, etc. are not meant to be used as backup solutions, and I shouldn't be feeding big tech monsters with more data. Especially personal data.

So, I decided to break the inertia and set up my own backup solution. It won't be perfect, but it will be simple, cheap, useful and the most important: fun to build.

This is the first post of a series where I intend to share my quest to establish a minimal, reliable solution to streamline my backup routine. 

## What do I need to back up?

First, we need to understand my reality and what I do with my data. First of all, as a software engineer, I have... 

- **A bunch of personal projects**: most of them unfinished, but full of useful snippets and ideas.
- **One endless library of books**: I haven’t read 1/3 of them, but they are precious to me.
- **Personal documents**: digital copies of personal documents that I often use while dealing with my daily life problems and bureaucracy.
- **Photos & Videos**: the usual phone clutter that holds special moments of our lifes.

We can categorize this data by availability into 2 sets:

### Data that I (may) need anywhere.
Personal documents, photos, and videos. I want quick access to these, even outside home, and they’re already synced across devices via my cloud provider.  

### Data I only need at home
Projects, books, notes. These don’t need to be synced to my phone or tablet, and in fact, syncing code to cloud folders has caused me issues before.  

Great! Based on that, I can say that I mainly need **two storage spaces**: 
1. A network-shared folder for data that doesn't belong in the cloud.
2. A cold storage area for copies of everything, including what I pull down from the cloud.

![Backup Landscape Diagram - Step 1](/images/backup/backup_landscape_step1.png)

## The 3-2-1 backup rule

The classic principle for reliable backups:  

1. **3 copies of data** — the original plus two backups.  
2. **2 different media types** — not all eggs in one basket.  
3. **1 off-site copy** — in case of fire, theft, or disaster.  

Here’s how I’m mapping it:  

| Rule | Data Synced Across Devices | Other Data |
|------|-----------------------------|------------|
| 3 copies | Cloud Storage, NAS Drive, Local Backup Media | Laptop Drive, NAS Drive, Local Backup Media |
| 2 different media | Local Backup Media, AWS S3 | Local Backup Media, AWS S3 |
| 1 off-site | AWS S3 | AWS S3 |

For off-site, I’m using **AWS S3 Glacier Deep Archive**. Cheap, reliable, and fits my low-cost goal.  

## How often?

Frequency is always a trade-off between cost and recoverability. My routine looks like this:  

- I work 9–6, and outside that I code, study, or write.  
- Losing 6 hours of data would be unpleasant, but survivable.  
- My dataset is ~17GB compressed, with ~100MB changes every 6 hours.  

Based on this, I chose:  

- **1 full backup monthly** (17GB).  
- **1 differential backup every 6 hours** (~12GB total/month).  

That’s ~30GB/month. Add 20% margin → **~36GB/month**.  
A 500GB disk holds over 12 months, which is plenty.  

![Backup Landscape Diagram - Final](/images/backup/backup_landscape_step2.png)

## Hardware setup

I am planning to initially use components that I already have at home:

- **Raspberry Pi 4B (2GB RAM)** → runs the automation.  
- **256GB NVMe SSD (USB3 hat)** → network-shared folder + cloud sync mirror.  
- **1TB USB HDD** → backup destination. Low-cost, flexible. (Eventually I’ll upgrade to a dock with proper cooling and reliability.)  


## Software stack

| Component | Purpose |
|-----------|---------|
| Raspberry OS Lite | Operating system |
| SSH | Remote access |
| SAMBA | Network file sharing |
| CRON | Schedule backups |
| UFW | Firewall |
| OneDrive Client for Linux | Pull data from OneDrive |
| TAR + GZIP | Archive + compress |
| OpenSSL | Encrypt before upload |
| AWS CLI | Upload to S3 |

On top of that, I’ll harden access and automate as much as possible.  


## Off-site storage cost

At the time of writing, AWS S3 Glacier Deep Archive costs **$0.00099/GB/month**. With I quick and easy math we can see how a bargain it will be:

```
Storage Requirement: ~ 36GB / * 12 months ~ 432GB for one year of backup retentnion
Monthly Cost: USD 0.00099 per GB per month * 432 GB (after 12 months of backup) = USD 0.43 per month
Yearly Cost: USD 0.43 * 12 months = USD 5.16
```

For less than **$6/year**, I can have a reliable, long-term off-site storage for my humble backup.

## Wrapping up

With ~36GB/month of backups, a Raspberry Pi, and some spare drives, I can implement a **3-2-1 strategy** that’s cheap, reliable, and good enough for a regular person like me.  

Next, I’ll share the configuration details — but I won’t just copy-paste one of the 2,000 Raspberry Pi tutorials out there. Expect something practical and personal.   