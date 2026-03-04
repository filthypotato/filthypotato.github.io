---
title: "Recovering Files From a Corrupted External Drive Using ddrescue, TestDisk, and PhotoRec"
date: 2026-03-09 08:00:00 -0800
categories: [Homelab, Digital Forensics]
tags: [Forensics, Data Recovery, Linux, Photorec, Testdisk, Ddrescue]
image:
  path: /assets/img/test-disk-banner.webp
  alt: Testdisk showing mismatch sectors and heads/cylinder
---

> This recovery focused on filesystem corruption and file carving.
> If you're interested in recovering deleted partitions instead, check out my previous investigation:
>
> [Partition Recovery with Autopsy + FTK Imager](/posts/partition-recovery-autopsy-ftk-imager/)

{:toc}

## The Problem

Recently one of my external hard drives decided to betray me.

The drive had a huge collection of books on it that my wife had been storing. When I plugged it into my PC… **everything was gone.** Along with 300GBs of retro games from SNES all the way to PS2.

No folders.

No filenames.

Just empty space where the data used to be.

![Empty External Drive](/assets/img/empty-external-drive.webp)

Instead of panicking and randomly copying files around (which can actually make recovery worse), I decided to treat the situation like a **small digital forensics investigation**.

---

## Preserve the Original Evidence

The first rule of digital forensics:

> Never work directly on the original evidence.

Drilling through a drive, mounting it, or copying files incorrectly can **overwrite sectors that still contain recoverable data**.

So before doing anything else, I created a **forensic copy of the damaged disk** and performed all recovery work on that copy instead.

This allows me to:

- Preserve the original drive exactly as it is
- Safely experiment with recovery tools
- Restart the recovery process if something goes wrong
- Prevent accidental data destruction

This is the same methodology used in professional digital forensics investigations.

---

## Creating a Forensic Disk Image

The first step was creating a full disk image of the damaged drive.

For this I used **ddrescue**, which is designed for copying failing or damaged drives safely.

Unlike normal copy tools, `ddrescue`:

- Skips unreadable sectors
- Logs progress
- Allows recovery attempts to resume later

Example command:

```bash
sudo ddrescue -f -n /dev/sde external_drive.img external_drive.log
```

This created a **raw disk image** that I could safely analyze without touching the original drive.

---

## Verifying the Disk Image

After creating the disk image, I generated a SHA256 hash to document the integrity of the forensic copy.

```bash
sha256sum bookdrive.img
```

Output:

`7b5c4d3fef360881bd91a6748d42c06e322a1678dea905e9bd553fa8a8cda124  bookdrive.img`

## A Common Misconception about Deleted Files

Most people think when something is deleted from a computer or drive… that it’s gone forever.

In reality, that’s almost never the case.

When a file is deleted, the operating system usually **doesn't erase the actual data immediately**. Instead, it simply removes the reference to that file from the filesystem and marks that space as available for reuse.

The data itself often remains on the disk until something else eventually overwrites it.

This is exactly why digital forensics tools are able to recover files long after they appear to be gone.

---

## Investigating the Filesystem with TestDisk

Once the disk image was created, I used **TestDisk** to examine the partition table and filesystem structure.

TestDisk is extremely useful for:

- Recovering lost partitions
- Rebuilding damaged partition tables
- Identifying filesystem corruption

Command used:

```bash
sudo testdisk external_drive.img
```

Unfortunately in this case the filesystem structure appeared to be severely damaged, so direct filesystem recovery wasn't possible.

That meant it was time to move on to **file carving**.

---

## File Carving with PhotoRec

Since the filesystem metadata was damaged, I used **PhotoRec** to scan the disk image sector-by-sector looking for recoverable files.

PhotoRec works by identifying **file signatures** instead of relying on the filesystem. While Photorec and TestDisk are not hardware write blockers, but they are designed to operate in a read only manner to prevent data modification during recovery.

That means it can recover files even when:

- Filenames are lost
- Folder structures are gone
- The filesystem is completely corrupted

Command used:

```bash
sudo photorec external_drive.img
```

PhotoRec then began scanning the disk and recovering files based purely on raw data signatures.

---

## Recovery Progress

The scan began discovering files almost immediately.

So far the recovery process has identified thousands of recoverable files including:

- ZIP archives
- PDFs
- Images
- Media files

Example recovery output:

- ZIP files recovered
- MP3 files recovered
- PDF documents recovered

Photorec had found 6000+ files in 4 hours. Even though some time still remained, this goes to show what is possible. Just remember, this drive had 0 visible files but with some digital forensics we can accomplish recovering deleted files.

![Photorec finding 6000+ files](/assets/img/photorec.webp)

---

## Why File Names Are Missing

One thing that surprises people during recovery is that **filenames and folders are usually lost**.

That's because filenames are stored in filesystem metadata.

When that metadata becomes corrupted, tools like PhotoRec can still recover the raw files — but they cannot reconstruct the original directory structure.

So recovered files typically appear as:

```
recup_dir.1/
recup_dir.2/
recup_dir.3/
```

With filenames like:

```
f12345678.pdf
f22345671.zip
```

![Some of the files that have been recovered so far!](/assets/img/some-recovered-files.webp)

Sorting and rebuilding the collection becomes part of the recovery process.

---

## Final Recovery Results

After letting PhotoRec run for several hours, the scan ended up recovering **over 50GB of data** from the damaged drive.

Most of the recovered files were exactly what I was hoping to find — **the book collection that had seemingly disappeared.**

In total the recovery process produced:

- **1,160 recovered files**
- **About 50.7GB of data**
- Mostly **PDF and archive files**

Even though the original filenames and folder structure were lost, the actual content of the files survived.

![Recovered book collection folder](/assets/img/books-recovered-folder.webp)

Once I sorted through the recovered files, it became clear that **most of the original collection had survived the corruption.**

What originally looked like a completely wiped drive turned out to be **a recoverable filesystem failure**.

---

## Why This Matters

This situation is a perfect example of something many people misunderstand about data loss.

Just because files **disappear from the filesystem** does not mean the data itself is gone.

Until those sectors are overwritten, forensic recovery tools can often still reconstruct the files directly from the raw disk.

This is why forensic investigators are able to recover evidence from drives that appear empty.

---

## Lessons Learned

While the recovery was successful, the experience reinforced several important lessons:

- Always maintain backups of important data
- Never perform recovery directly on the original disk
- Creating a disk image first protects the original evidence
- File carving tools can recover data even when filesystems are destroyed

> **If data only exists in one place, it’s already at risk.**

From now on this collection will exist in **multiple locations**, not just a single external drive.

Because sometimes recovery works…

…and sometimes it doesn't.

---

*If it's not broken, fix it til it is.*


