# Payload Collections (Web & Network Security)

This repository contains curated payloads and wordlists for offensive security testing. It is organized into:

- `web_payloads/` – web application vulnerabilities (XSS, SQLi, SSRF, etc.)
- `network_payloads/` – network, protocol and port‑specific attacks
- `common_payloads/` – shared wordlists (passwords, usernames, JWT secrets, tool‑specific lists)
- `number/` – numeric wordlists for PIN/OTP and code brute‑forcing

Additional documentation:
- `web_payloads/README.md` – detailed web vulnerability overview and quick reference
- `network_payloads/README.md` – network attacks overview and examples
- `common_payloads/README.md` – shared wordlists and how to use them
- `number/README.md` – numeric lists for PIN/OTP brute‑forcing

## Quick Start

```bash
# List available web vulnerability categories
ls web_payloads/

# Example: inspect SQL Injection payloads
cd "web_payloads/SQL Injection"
cat payloads.txt

# Example: inspect port‑specific network payloads
cd "network_payloads/Port_Specific_Attacks"
cat payloads.txt
```

Payload files are plain text, so you can use your usual tooling:

```bash
# Search for a keyword across all payloads
grep -R "UNION SELECT" web_payloads/

# Pipe into fuzzing/bruteforce tools
ffuf -w "web_payloads/SQL Injection/payloads.txt" -u https://target/FUZZ
```

## Ethics & Legal

- For **educational and authorized testing only**
- Do **not** use against systems you do not own or have explicit written permission to test

## Credits

Many payloads are adapted from or inspired by:

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- Other open‑source security resources referenced in the sub‑folder READMEs
