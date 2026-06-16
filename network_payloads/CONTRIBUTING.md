# Contributing to Network Security Payloads

Thank you for your interest in contributing! This document provides guidelines for adding new payloads, attack techniques, and improving existing content.

## Table of Contents

- [How to Contribute](#how-to-contribute)
- [Repository Structure](#repository-structure)
- [Content Guidelines](#content-guidelines)
- [Submission Process](#submission-process)
- [Testing Requirements](#testing-requirements)

---

## How to Contribute

We welcome contributions in the following areas:

1. **New Attack Payloads** - Add commands and techniques
2. **Tool Guides** - Installation and usage instructions
3. **Bypass Techniques** - Methods to evade security controls
4. **Defense Strategies** - Mitigation and detection methods
5. **Port-Specific Guides** - Comprehensive port attack guides
6. **Bug Fixes** - Corrections to existing content
7. **Documentation** - Improve clarity and examples

---

## Repository Structure

```
network_payloads/
├── README.md                          # Main repository overview
├── TOOLS_REFERENCE.md                 # Tool installation guides
├── ATTACK_METHODOLOGY.md              # Complete attack workflow
├── QUICK_REFERENCE.md                 # Cheat sheet
├── CONTRIBUTING.md                    # This file
│
├── Network_Reconnaissance/
│   ├── README.md                      # Comprehensive guide
│   ├── payloads.txt                   # Commands and techniques
│   ├── tools.md                       # Tool-specific usage
│   └── bypass_techniques.md           # Evasion methods
│
├── Protocol_Attacks/
│   ├── README.md
│   ├── payloads.txt
│   └── ...
│
├── MITM_Attacks/
│   ├── README.md
│   ├── payloads.txt
│   └── ...
│
├── DoS_DDoS/
├── VPN_Attacks/
├── Wireless_Attacks/
├── Network_Evasion/
├── Routing_Attacks/
│
└── Port_Specific_Attacks/
    ├── Port_21_FTP/
    │   ├── README.md                  # Complete FTP attack guide
    │   ├── payloads.txt               # FTP-specific commands
    │   ├── bypass_techniques.md       # FTP security bypass
    │   └── defenses.md                # Defense recommendations
    ├── Port_22_SSH/
    ├── Port_25_SMTP/
    ├── Port_53_DNS/
    ├── Port_80_HTTP/
    ├── Port_139_445_SMB/
    └── ...
```

---

## Content Guidelines

### For payloads.txt Files

**Format**:
```bash
## Category Name

### Specific Technique
command here                                           # Brief description
another-command options target                         # What it does

### Another Technique
# Multi-line example
cat > script.sh << 'EOF'
#!/bin/bash
echo "Example script"
EOF
```

**Requirements**:
- Clear categorization with ## and ###
- Comments explaining what each command does
- Working examples (test before submitting)
- Align comments at column 70 for readability
- Include both basic and advanced techniques

### For README.md Files (Attack Guides)

**Required Sections**:

1. **Overview** - Protocol/service description
   ```markdown
   ## Overview
   **Protocol**: Name
   **Port**: Number
   **Transport**: TCP/UDP
   **Encryption**: Yes/No
   ```

2. **Attack Objectives** - What you're trying to achieve
3. **Attack Methodology** - Step-by-step attack process
4. **Tool Selection Guide** - When to use which tool
5. **Bypass Techniques** - How to evade security
6. **Information Extraction** - What data to extract
7. **Practical Scenarios** - Real-world examples
8. **Security Recommendations** - Defense strategies
9. **Common Mistakes** - What to avoid
10. **Tools Summary** - Best tool for each task

**Example Structure**:
```markdown
# Port XX - Service Name - Complete Attack Guide

## Overview
[Service description]

## Attack Objectives
- Objective 1
- Objective 2

## Attack Methodology

### Phase 1: Discovery
[Commands and explanation]

### Phase 2: Exploitation
[Detailed attack steps]

## Bypass Techniques

### Bypassing XXX
[How to bypass security control]

## Information Extraction
[What information can be extracted and how]

## Practical Attack Scenario
[Complete walkthrough]

## Security Recommendations
[Defense strategies]
```

### For Bypass Techniques

**Structure**:
```markdown
## Defense Mechanism Name

**What it is**: Brief description
**How it works**: Technical explanation
**Bypass Method**: Step-by-step bypass

**Example**:
```bash
# Commands to bypass
```

**When to Use**: Scenarios where this applies
**Success Rate**: High/Medium/Low
**Detection Risk**: High/Medium/Low
```

### Code Style

**Shell Commands**:
- Use `bash` syntax
- Include error handling where appropriate
- Test commands before submitting
- Use full paths for binaries when important

**Python Scripts**:
```python
#!/usr/bin/env python3
# Description of what script does

import required_modules

def main():
    # Main logic here
    pass

if __name__ == "__main__":
    main()
```

**Comment Style**:
```bash
# Good: Explains WHY
nmap -sS target  # Stealth scan to avoid detection

# Bad: Explains WHAT (obvious from command)
nmap -sS target  # Run nmap with -sS flag
```

---

## Submission Process

### 1. Fork the Repository
```bash
git clone https://github.com/yourusername/network_payloads
cd network_payloads
git checkout -b feature/new-attack-technique
```

### 2. Make Your Changes
- Add your content following the structure above
- Test all commands in a lab environment
- Update relevant README files
- Add examples and explanations

### 3. Test Your Content
- Verify all commands work
- Check markdown formatting
- Ensure no sensitive information included
- Test on multiple OS if applicable (Linux/Windows/macOS)

### 4. Commit Your Changes
```bash
git add .
git commit -m "Add: Port 3306 MySQL attack guide"
```

**Commit Message Format**:
- `Add:` New content
- `Update:` Modify existing content
- `Fix:` Bug fixes or corrections
- `Docs:` Documentation improvements

### 5. Create Pull Request
- Clear title describing the addition
- Detailed description of what was added
- Reference any related issues
- Include test results if applicable

---

## Testing Requirements

### Before Submitting

**Required Checks**:
- [ ] All commands tested in lab environment
- [ ] No real targets/IPs in examples (use 192.168.1.x, example.com)
- [ ] No personally identifiable information (PII)
- [ ] No actual credentials or API keys
- [ ] Markdown formatting validated
- [ ] Links work and point to correct resources
- [ ] Screenshots included where helpful (optional)

**Testing Environments**:
- Virtual machines (recommended)
- Isolated lab networks
- Intentionally vulnerable machines (DVWA, HackTheBox, etc.)

**DO NOT**:
- Test on production systems
- Test without authorization
- Include illegal content
- Include malicious payloads without explanation

---

## Checklist for New Port-Specific Guide

When adding a new port guide, ensure it includes:

- [ ] Overview section (protocol, port, transport)
- [ ] Attack objectives
- [ ] Discovery and reconnaissance commands
- [ ] Banner grabbing techniques
- [ ] Service enumeration
- [ ] At least 3 exploitation techniques
- [ ] Brute force methods (if applicable)
- [ ] Bypass techniques for common defenses
- [ ] Information extraction methods
- [ ] Practical attack scenario (complete walkthrough)
- [ ] Security recommendations for defenders
- [ ] Common mistakes section
- [ ] Tools summary table
- [ ] Related attacks/ports

---

## Content Priorities

**High Priority** (Most Needed):
- Port-specific attack guides (23, 25, 53, 80, 139/445, 443, 1433, 3306, 3389, 5432)
- Defense/mitigation techniques
- Bypass methods for modern security controls
- Real-world attack scenarios
- Tool usage examples

**Medium Priority**:
- Additional protocol attacks
- Evasion techniques
- Automation scripts
- Advanced exploitation

**Low Priority**:
- Theoretical content
- Deprecated techniques
- Rare protocols

---

## Review Criteria

Pull requests will be evaluated on:

1. **Accuracy** - Commands work as described
2. **Completeness** - Follows structure guidelines
3. **Clarity** - Easy to understand and follow
4. **Safety** - No dangerous/illegal content
5. **Originality** - Not duplicating existing content
6. **Testing** - Evidence of testing provided
7. **Documentation** - Well-commented and explained

---

## Recognition

Contributors will be:
- Listed in repository contributors
- Credited in relevant files
- Mentioned in release notes

---

## Questions?

If you have questions about contributing:
- Open an issue with the `question` label
- Check existing issues and discussions
- Review existing guides for examples

---

## Legal Notice

By contributing, you agree that:
- Content is for authorized security testing only
- You have tested in legal environments
- No malicious intent or illegal activity
- Content follows responsible disclosure principles

---

## Resources for Contributors

**Learning Resources**:
- OWASP Testing Guide
- PTES (Penetration Testing Execution Standard)
- NIST Cybersecurity Framework
- MITRE ATT&CK Framework

**Testing Environments**:
- VulnHub (free vulnerable VMs)
- HackTheBox (practice platform)
- TryHackMe (guided labs)
- DVWA (Damn Vulnerable Web App)
- Metasploitable (intentionally vulnerable Linux)

**Markdown Editors**:
- VSCode with Markdown Preview
- Typora
- Mark Text
- Online: Dillinger.io

---

Thank you for contributing to the network security community!
