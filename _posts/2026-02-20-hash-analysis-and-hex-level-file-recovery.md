---
title: "Case Study: USB File Invemtory, Hash Comparison, and Hex-Level Data Recovery"
date: 2026-01-15 12:00:00 -0800
categories: [Digital Forensics]
tags: [Windows Forensics, Command Line, Hash Comparison, Hex Editor, File Carving]
image:
  path: "/assets/img/hash-analysis-banner.png"
  alt: Digital forensic command line and hex editor analysis
---

## Case Overview

In this lab scenario, I was provided with a USB drive containing files relevant to a legal matter. Since returning to a lab environment was not possible, the objective was to generate a quick but thorough report directly from a Windows system.

The tasks included:

- Performing a full file inventory using command line tools
- Identifying hidden files
- Comparing similar files to detect differences
- Reviewing hex editor functionality from a forensic perspective
- Recovering an embedded JPEG file using hexadecimal signatures

All work was performed on Windows 11.

---

## USB File Inventory Using Command Line

During this setup, the listing shows files inside of a thumb drive. All files were located and listed as visible files with timestamps, there were no hidden or invisible files found.

To generate the report, I used the `dir` command with multiple switches:

- `/s` to include subdirectories
- `/t:w` to display last modified timestamps
- `/a:h` to display hidden files
- `/b` to display files by path

This allowed me to generate:

- Visible files list
- Hidden files list
- Visible files listed by full path
- Hidden files listed by full path

While reviewing the directory output, I identified duplicate documents located in different directories:

- THE FREEDOM OF INFORMATION ACT.docx
- RevHistory.xlsx

Both appeared in the root directory and inside a subfolder named `Data`, with different timestamps. This suggests that the files were copied or moved instead of being hidden.

### File Comparison Using COMP and FC

An attorney provided two files that appeared identical. The objective was to determine whether any data differences existed.

I used the Windows `comp` command:

```bash
comp /m Summary1.csv Summary2.csv
```

**The output revealed the numeric differences between the two files. To further identify the exact differencesm I used:

```bash
fc Summary1.csv Summary2.csv
```

Two of the differences that were identified were:

  - "1,041" → "<span style="color:#ff4d4d;">9</span>,041"

  - "-163,768" → "-163,76<span style="color:#ff4d4d;">9</span>"

This shows how even small numerical changed can be detected using Windows tools.

## Extracting an Embedded JPEG with HxD

### What I did:

1. Opened **HxD**.
2. Went to **File > Open** and selected the data file I was given (`Hex_Extract.dat`).
3. Used **Search > Find** and switched to the **Hex-values** tab.
4. Searched for the JPEG header by typing:

   - `FFD8`

   Once it hit, HxD showed the start offset at:

   - `Offset(h): 1E5`

5. Then I searched again for the JPEG end marker:

   - `FFD9`

   HxD displayed the ending offset as `E947`, but because the marker is **two bytes**, the real end offset for the block is:

   - `E948`



### Selecting the Correct Byte Range

6. I used **Edit > Select block** and entered:

   - Start offset: `1E5`
   - End offset: `E948`

7. Then I copied the selected block (**Edit > Copy**).
8. Created a new blank file (**File > New**).
9. Pasted the selection using **Edit > Paste insert** (confirmed the prompt).

### Saving and Testing the Recovery

10. Saved the new file using **File > Save as** and gave it a `.jpg` extension.
11. Opened the saved `.jpg` in the default photo viewer to verify it worked.

The recovered image opened successfully (boat at a dock), which confirmed the extraction worked and the offsets were correct.

### Screenshot Suggestions

- HxD showing the `FFD8` hit and the start offset (`1E5`)
- HxD showing the `FFD9` hit and the note about the end offset (`E948`)
- Select block window showing start/end offsets
- Recovered image opened in Photos (only if it’s safe to share)


## Conclusion

This was my first type of Forensic Analysis I have done and it taught me a lot.

This exercise reinforced how powerful low level analysis can be in a forensic investigation. Even when a file is not visible through normal file system browsing, its raw data can still exist within a container file.

By identifying the JPEG header (`FF D8`) and footer (`FF D9`), calculating accurate offsets, and extracting the byte range manually, I was able to recover the embedded image successfully.

This kind of manual file carving helps build a deeper understanding of how forensic tools automate recovery behind the scenes. Instead of relying entirely on software, working directly with offsets and hex values strengthens foundational DFIR skills.
