# Included Wordlists

This repository now includes several well‑known wordlists locally under `wordlists/` so you don't have to rely on `/usr/share/wordlists`.

## Passwords

- `wordlists/passwords/rockyou.txt.tar.gz`  
  Compressed RockYou password list from the SecLists project (Git‑friendly, <100MB). Decompress after cloning.
- `wordlists/passwords/top-passwords-shortlist.txt`  
  Very small list of the most common passwords.
- `wordlists/passwords/probable-v2-wpa-top62.txt`  
  Small WiFi/WPA‑focused password shortlist.

After cloning, decompress RockYou (if you want the full list):

```bash
cd wordlists/passwords
tar -xzf rockyou.txt.tar.gz  # creates rockyou.txt (ignored by git)
```

## Directories (Web Content)

From the SecLists `Discovery/Web-Content` collection:

- `wordlists/directories/directory-list-2.3-small.txt`  
  Classic DirBuster priority‑ordered list (small).
- `wordlists/directories/directory-list-2.3-medium.txt`  
  Classic DirBuster medium list.
- `wordlists/directories/raft-small-directories.txt`  
  RAFT small directory wordlist.

All of these were downloaded from: https://github.com/danielmiessler/SecLists

## Usernames

- `wordlists/usernames/top-usernames-shortlist.txt`  
  Small list of very common usernames and default system/cloud users.

## Usage Examples

```bash
# ffuf with included directory list
ffuf -w wordlists/directories/directory-list-2.3-small.txt \
     -u "https://target/FUZZ" -mc 200,302

# Hydra/hashcat with rockyou
hydra -L common_payloads/username.txt -P wordlists/passwords/rockyou.txt ssh://192.168.1.10
```

See also:
- `common_payloads/README.md` for small, curated lists
- `common_payloads/WORDLIST_SOURCES.md` for update/refresh guidance
