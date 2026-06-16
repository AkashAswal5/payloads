## Common Payloads and Wordlists

This directory groups reusable wordlists and helper files that support both web and network testing.

- `fsocity.dic` – large wordlist from the Mr. Robot CTF; useful for directory brute‑forcing and password attacks
- `password.txt` – common passwords
- `username.txt` – common usernames
- `wordlist.txt` – general‑purpose mixed wordlist
- `1-4_all_letters_a-z.txt` – short alphabetic strings (1–4 letters) for quick fuzzing
- `common.txt` – common English words
- `All_attack.txt` – mixed payloads covering multiple vulnerability types
- `sql_injection_payload.txt` – basic SQL injection test strings
- `SQL.txt` – additional SQL‑related payloads/keywords
- `xss.txt` – common XSS test payloads
- `Traversal.txt` – path traversal strings
- `XML.txt` – XML/XXE‑related payloads
- `bad_chars.txt` – characters often filtered or dangerous in various contexts
- `dnsmap.txt`, `nmap.lst`, `fasttrack.txt`, `john.lst` – wordlists tuned for specific tools
- `jwt.secrets.list` – common JWT HMAC secrets
- `ffuf_dirs_small.txt` – small, high‑signal directory list tuned for ffuf/dir fuzzing
- `common_params.txt` – common HTTP parameter names for Burp Intruder, ffuf, wfuzz, etc.
- `hashcat_masks_basic.txt` – a few common hashcat masks for quick mask‑based attacks
 - `headers_common.txt` – common HTTP header names for header fuzzing and access‑control bypass testing
 - `reverse_shells.txt` – common multi-language reverse shell one‑liners (bash, nc, python, php, perl, PowerShell)

You can plug these files directly into tools such as Hydra, Burp, wfuzz, ffuf, or hash‑cracking utilities.

For **large, official wordlists** (e.g. `rockyou.txt`, big directory lists for ffuf/dirb/gobuster), see:

- `common_payloads/WORDLIST_SOURCES.md` – where to get RockYou, SecLists, and ffuf wordlists and how to combine them with this repo.

Example (ffuf directory fuzzing):

```bash
ffuf -w common_payloads/ffuf_dirs_small.txt -u "https://target/FUZZ" -mc 200,302
```

Example (hashcat mask attack, 6‑digit PIN):

```bash
hashcat -a 3 -m 0 hashes.txt ?d?d?d?d?d?d
```

Example (ffuf header fuzzing with common headers):

```bash
ffuf -w common_payloads/headers_common.txt -H "FUZZ: test" -u "https://target/" -mc all
```
