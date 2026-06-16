# Wordlists in This Repo and External Sources

This repository ships with:

- **Small, curated wordlists** in `common_payloads/`
- **Larger, common wordlists** directly in this repo under `wordlists/` (e.g. RockYou archive, popular directory lists, top usernames/passwords, WiFi lists)

Below are the included lists and the upstream projects they come from, plus how to update or extend them.

---

## 1. RockYou password list (included locally)

Widely used leaked password list.

This repo already includes a compressed version at:

- `wordlists/passwords/rockyou.txt.tar.gz`

After cloning, decompress it once:

```bash
cd wordlists/passwords
tar -xzf rockyou.txt.tar.gz  # creates rockyou.txt (ignored by git)
```

You can then use `rockyou.txt` directly with Hydra, hashcat, aircrack‑ng, etc. For updates, this file was downloaded from the **SecLists** project (see below).

---

## 2. SecLists (upstream source)

[SecLists](https://github.com/danielmiessler/SecLists) is the de‑facto standard collection of:

- Directory and file wordlists
- Password lists (including rockyou variants)
- Fuzzing payloads
- Discovery paths, usernames, etc.

Clone it locally:

```bash
git clone https://github.com/danielmiessler/SecLists.git ~/SecLists
```

Useful SecLists paths mirrored into this repo:

**Directories (Discovery/Web-Content):**
- `directory-list-2.3-small.txt`   → `wordlists/directories/directory-list-2.3-small.txt`
- `directory-list-2.3-medium.txt`  → `wordlists/directories/directory-list-2.3-medium.txt`
- `raft-small-directories.txt`     → `wordlists/directories/raft-small-directories.txt`

**Usernames (Usernames):**
- `top-usernames-shortlist.txt`    → `wordlists/usernames/top-usernames-shortlist.txt`

**Passwords (Common-Credentials & WiFi-WPA):**
- `top-passwords-shortlist.txt`    → `wordlists/passwords/top-passwords-shortlist.txt`
- `probable-v2-wpa-top62.txt`      → `wordlists/passwords/probable-v2-wpa-top62.txt`

Example usage with ffuf (using the **local** copies):

```bash
ffuf -w wordlists/directories/directory-list-2.3-small.txt \
     -u "https://target/FUZZ" -mc 200,302
```

---

## 3. ffuf and other tools

Tools like `ffuf`, `dirsearch`, `gobuster`, etc. can all use the lists in:

- `wordlists/directories/` (large, official-style lists)
- `common_payloads/ffuf_dirs_small.txt` (fast, curated list)

---

## 4. Combining this repo with external lists

Some common patterns:

- **Start small**: use `common_payloads/ffuf_dirs_small.txt` for fast discovery.
  - **Go deeper**: switch to the large directory lists in `wordlists/directories/` when you need coverage.
- **Passwords**: combine `common_payloads/password.txt` for quick checks with `wordlists/passwords/rockyou.txt` for heavier attacks.

Examples:

```bash
# Quick, targeted directory brute force
ffuf -w common_payloads/ffuf_dirs_small.txt -u "https://target/FUZZ" -mc 200,302

# Full scan with included directory list
ffuf -w wordlists/directories/directory-list-2.3-medium.txt \
     -u "https://target/FUZZ" -mc 200,302
```
