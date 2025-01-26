---
layout: post
title: "Security Rocks"
subtitle: "Easy Forensics Chall for TUCTF 2025"
author: "jxkxl"
header-style: text
tags:
  - CTF
  - Writeup
  - Forensics
  - TUCTF2025
---

## Description

I shared a super secret message, I hope it's secure.

## Tools Used

- Wireshark
- aircrack-ng
- CyberChef

## Approach

### Initial Analysis

Opening "dump-05.cap" in WireShark revealed numerous 802.11 packets transmitted among 4 or 5 devices, but no packets contained meaningful information.

### Enumeration

I inspected the protocol hierarchy and found the "802.1X" Authentication protocol listed. Filtering for this protocol uncovered 8 EAPOL packets, indicating that I could exploit the 4-way handshake to obtain the network passkey.

![[../assets/securityRocks1.png]]

### Exploitation

 **Step 1: Crack the Key**
Executed the command:
 
```bash
    aircrack-ng -z -w /usr/share/wordlists/rockyou.txt ~/CTF/TUCTF24/forensics/dump-05.cap
```

This allowed me to crack the hash and reveal the plaintext password: `youwontguessit92`.

![[../assets/securityRocks2.png]]

**Step 2: Decrypt Traffic**
Set the obtained password as the WPA decryption key in WireShark under `Preferences > IEEE 802.11 > Decryption Keys` as `wpa-pwd`.

This made the traffic much clearer, revealing FTP data being transferred.

**Step 3: Export FTP Objects**
Exported the FTP objects, which included the file "secret.txt".

Upon opening, it contained the message:  
"Here's my super secret flag, I made it extra secure ;) 1KZTi2ZV7tO6yNxslvQbjRGL54BsPVyskwv4QaR29UMKj".
 
**Step 4: Decode the Flag**
Input the encoded string into CyberChef and decoded it using Base62.

![[../assets/securityRocks3.png]]

The result was the flag:   `TUCTF{w1f1_15_d3f1n173ly_53cure3}`.

## Solution

- **Flag:** `TUCTF{w1f1_15_d3f1n173ly_53cure3}`