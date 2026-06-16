# Numeric Wordlists

This folder contains numeric wordlists useful for PIN, OTP, and code brute‑forcing in CTFs and lab environments.

- `3-digits-0000-9999.txt` – zero‑padded numbers in a small range
- `4-digits-0000-9999.txt` – all 4‑digit codes from 0000 to 9999
- `5-digits-00000-99999.txt` – all 5‑digit codes
- `6-digits-000000-999999.txt` – all 6‑digit codes

Use them with tools like Hydra, Burp Intruder, ffuf, wfuzz, or custom scripts.

```bash
# Example: brute‑force a 4‑digit PIN parameter
ffuf -w 4-digits-0000-9999.txt -u "https://target/login?pin=FUZZ"
```
