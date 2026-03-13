# 🔐 Multi-Account GitHub Setup

## Personal Owner Account + Organization Copilot Account

**Author:** Rei Reltroner

**Environment:** Windows Development Workstation

```
OS: Windows 11
Shell: Windows PowerShell
Git: Git for Windows
Editor: VS Code
Node: Node.js (development runtime)
```

---

# I. Overview

This document describes the **deterministic setup for using multiple GitHub accounts on a single development machine**.

The setup supports two independent identities:

1. **Personal GitHub account** (repository owner)
2. **Organization GitHub account** (Copilot-enabled contributor)

Without proper configuration, Git credential conflicts may occur during repository operations such as `git push`.

This document defines a **stable SSH multi-account architecture** that isolates credentials per account.

---

# II. Account Roles

## 1️⃣ Personal Owner Account

```
Username: Reltroner
Role: Repository Owner
```

Purpose:

* Personal projects
* Solo development
* Repository ownership

Example repository:

```
https://github.com/Reltroner/reltroner-studio
```

---

## 2️⃣ Organization Contributor Account

```
Username: reltronersk
Role: Organization Contributor
```

Purpose:

* Contribution to organization repositories
* Access to **GitHub Copilot Pro (organization license)**
* Collaborative development

---

# III. Problem Description

When multiple GitHub accounts are used on the same machine, Git may reuse a previously stored credential.

Example error:

```
remote: Permission to Reltroner/reltroner-studio.git denied to reltronersk
fatal: 403
```

Root cause:

```
Git Credential Manager stores one global identity
```

This leads to:

```
Organization account attempting to push to personal repository
```

---

# IV. Solution Strategy

The recommended professional solution is:

> **SSH multi-account architecture**

Each GitHub account receives its own SSH key.

```
GitHub Account A → SSH Key A
GitHub Account B → SSH Key B
Repository selects which key to use
```

Architecture layout:

```
~/.ssh
 ├── id_ed25519_reltroner
 ├── id_ed25519_reltronersk
 └── config
```

---

# V. Generating SSH Keys

Open PowerShell:

```
PS C:\Users\Reltropolis>
```

### Generate SSH key for personal account

```bash
ssh-keygen -t ed25519 -C "reltroner@github" -f C:\Users\Reltropolis\.ssh\id_ed25519_reltroner
```

Output:

```
Your identification has been saved in:
C:\Users\Reltropolis\.ssh\id_ed25519_reltroner

Your public key has been saved in:
C:\Users\Reltropolis\.ssh\id_ed25519_reltroner.pub
```

---

### Generate SSH key for organization account

```bash
ssh-keygen -t ed25519 -C "reltronersk@github" -f C:\Users\Reltropolis\.ssh\id_ed25519_reltronersk
```

Generated files:

```
id_ed25519_reltronersk
id_ed25519_reltronersk.pub
```

---

# VI. Upload SSH Keys to GitHub

## Personal account key

Display the key:

```bash
type C:\Users\Reltropolis\.ssh\id_ed25519_reltroner.pub
```

Copy the output and add it to:

```
https://github.com/settings/keys
```

Suggested title:

```
Reltroner Windows Workstation
```

---

## Organization account key

Display:

```bash
type C:\Users\Reltropolis\.ssh\id_ed25519_reltronersk.pub
```

Add it to the organization GitHub account:

```
https://github.com/settings/keys
```

---

# VII. SSH Multi-Account Configuration

Create SSH configuration file:

```
C:\Users\Reltropolis\.ssh\config
```

Add the following:

```
Host github-reltroner
    HostName github.com
    User git
    IdentityFile C:\Users\Reltropolis\.ssh\id_ed25519_reltroner

Host github-reltronersk
    HostName github.com
    User git
    IdentityFile C:\Users\Reltropolis\.ssh\id_ed25519_reltronersk
```

Alias meaning:

```
github-reltroner   → personal GitHub account
github-reltronersk → organization GitHub account
```

---

# VIII. Verifying SSH Authentication

Test personal account:

```bash
ssh -T git@github-reltroner
```

Expected output:

```
Hi Reltroner! You've successfully authenticated.
```

---

Test organization account:

```bash
ssh -T git@github-reltronersk
```

Expected output:

```
Hi Reltroner! You've successfully authenticated.
```

Note: GitHub intentionally does not provide shell access.

---

# IX. Converting Repository Remote to SSH

Navigate to your project directory.

Example:

```
C:\Projects\learn-nextjs
```

Check remote:

```bash
git remote -v
```

Example output:

```
https://github.com/Reltroner/reltroner-studio
```

Convert to SSH alias:

```bash
git remote set-url origin git@github-reltroner:Reltroner/reltroner-studio.git
```

Verify:

```bash
git remote -v
```

---

# X. Test Repository Push

Push changes:

```bash
git push origin main
```

Expected result:

```
Everything up-to-date
```

This confirms that the repository is authenticated using the **Reltroner SSH identity**.

---

# XI. Final Authentication Architecture

Repository operations:

```
Git Push
 ↓
SSH Key
 ↓
github-reltroner
 ↓
Reltroner account
```

Copilot access:

```
VS Code Login
 ↓
GitHub OAuth
 ↓
reltronersk account
```

Both systems operate independently.

---

# XII. Workflow Separation

## Personal Development Workflow

```
Account: Reltroner
Usage:
- personal repositories
- solo projects
- repository ownership
```

---

## Organization Development Workflow

```
Account: reltronersk
Usage:
- organization repositories
- GitHub Copilot Pro
- collaborative development
```

---

# XIII. Benefits

### Security

```
Personal repositories cannot be accessed using organization credentials.
```

---

### Stability

```
No Git Credential Manager conflicts.
```

---

### Scalability

This architecture supports additional accounts:

```
3 accounts
4 accounts
Enterprise GitHub environments
```

---

# XIV. Recommended Project Directory Layout

Example structure:

```
C:\Projects
 ├── Personal
 └── Organization
```

Mapping guideline:

```
Personal repositories → github-reltroner
Organization repositories → github-reltronersk
```

---

# XV. Optional Advanced Setup

For enterprise-scale development environments, consider:

* SSH agent management
* Repository-specific Git configuration
* Conditional Git configuration (`includeIf`)
* Hardware security keys (FIDO2)

---

# XVI. Final Status

System verified operational.

```
SSH authentication working
Git push working
Copilot access preserved
Multi-account conflict resolved
```

---

# XVII. Architecture Diagram

The following diagram illustrates how multiple GitHub accounts coexist on a single workstation using **SSH identity isolation**.

```
                ┌─────────────────────────────┐
                │      Developer Machine      │
                │      (Windows Workstation)  │
                └──────────────┬──────────────┘
                               │
                               │ Git Operations
                               │
                        ┌──────▼──────┐
                        │  Git Client │
                        └──────┬──────┘
                               │
                      SSH Identity Selection
                               │
            ┌──────────────────┴──────────────────┐
            │                                     │
            │                                     │
   ┌────────▼────────┐                   ┌────────▼────────┐
   │ SSH Identity A  │                   │ SSH Identity B  │
   │ id_ed25519_     │                   │ id_ed25519_     │
   │ reltroner       │                   │ reltronersk     │
   └────────┬────────┘                   └────────┬────────┘
            │                                     │
            │ SSH Alias                           │ SSH Alias
            │                                     │
     ┌──────▼──────┐                       ┌──────▼──────┐
     │ github-     │                       │ github-     │
     │ reltroner   │                       │ reltronersk │
     └──────┬──────┘                       └──────┬──────┘
            │                                     │
            │                                     │
   ┌────────▼────────┐                   ┌────────▼────────┐
   │ Personal GitHub │                   │ Organization    │
   │ Account         │                   │ GitHub Account  │
   │ Reltroner       │                   │ reltronersk     │
   └─────────────────┘                   └─────────────────┘
```

This architecture guarantees:

* deterministic credential selection
* complete identity separation
* zero credential manager conflicts

---

# XVIII. Troubleshooting Guide

If authentication problems occur, follow this diagnostic sequence.

### Step 1 — Verify SSH key files exist

```
C:\Users\<username>\.ssh
```

Expected:

```
id_ed25519_reltroner
id_ed25519_reltronersk
config
```

---

### Step 2 — Verify SSH config

Open:

```
~/.ssh/config
```

Confirm alias configuration exists and paths are correct.

---

### Step 3 — Test SSH connectivity

Run:

```bash
ssh -T git@github-reltroner
```

and

```bash
ssh -T git@github-reltronersk
```

Expected output:

```
Hi <username>! You've successfully authenticated.
```

---

### Step 4 — Check repository remote

Run:

```bash
git remote -v
```

Example expected output:

```
origin  git@github-reltroner:Reltroner/reltroner-studio.git
```

If HTTPS appears instead of SSH, authentication conflicts may occur.

---

### Step 5 — Verify Git identity

Check local configuration:

```bash
git config user.name
git config user.email
```

This affects commit metadata but **not authentication**.

---

# XIX. Common Failure Cases

Below are the most common issues encountered in multi-account Git setups.

---

## 1️⃣ Permission Denied (403)

Error:

```
remote: Permission to <repo> denied
fatal: 403
```

Cause:

```
Wrong GitHub account attempting to access repository
```

Example:

```
reltronersk trying to push to Reltroner repo
```

Fix:

Ensure remote URL uses the correct SSH alias:

```bash
git remote set-url origin git@github-reltroner:Reltroner/reltroner-studio.git
```

---

## 2️⃣ Permission Denied (publickey)

Error:

```
Permission denied (publickey).
```

Cause:

```
SSH key not registered on GitHub
```

Fix:

1. Copy SSH key:

```
type ~/.ssh/id_ed25519_reltroner.pub
```

2. Add to:

```
https://github.com/settings/keys
```

---

## 3️⃣ Wrong SSH Key Used

Symptom:

```
push attempts with wrong account
```

Diagnosis:

Run verbose SSH test:

```bash
ssh -vT git@github-reltroner
```

Check which identity file is used.

Expected log line:

```
Offering public key: id_ed25519_reltroner
```

---

## 4️⃣ HTTPS Remote Instead of SSH

Error often appears as credential prompt.

Check remote:

```bash
git remote -v
```

If output contains:

```
https://github.com/...
```

Convert to SSH:

```bash
git remote set-url origin git@github-reltroner:Reltroner/repo.git
```

---

## 5️⃣ SSH Config Ignored

Cause:

```
SSH config file named incorrectly
```

Ensure file name is exactly:

```
config
```

Not:

```
config.txt
config.conf
```

---

# XX. Enterprise Multi-Account Scaling Pattern

The architecture can scale to **multiple identities** across organizations.

Example enterprise scenario:

```
Personal account
Startup organization
Corporate organization
Open source maintainer account
```

---

## Example SSH Identity Layout

```
~/.ssh
 ├── id_ed25519_personal
 ├── id_ed25519_startup
 ├── id_ed25519_corporate
 ├── id_ed25519_open_source
 └── config
```

---

## Example SSH Config

```
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

Host github-startup
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_startup

Host github-corporate
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_corporate

Host github-oss
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_open_source
```

---

## Repository Mapping Strategy

Example directory structure:

```
C:\Projects
 ├── Personal
 ├── Startup
 ├── Corporate
 └── OpenSource
```

Each repository uses its corresponding SSH alias.

Example remote:

```
git@github-corporate:company/project.git
```

---

# XXI. Operational Best Practices

Recommended engineering practices:

### Identity Isolation

Never reuse SSH keys across accounts.

---

### Repository Ownership Discipline

Always verify:

```
repository owner == SSH alias identity
```

---

### Use SSH Instead of HTTPS

SSH avoids credential caching conflicts.

---

### Document Identity Architecture

Store the configuration in:

```
/docs/github-multi-account.md
```

So the setup can be replicated by other developers.

---

# XXII. Final Architecture State

The workstation now supports:

```
Multiple GitHub identities
Independent SSH authentication
Copilot organization access
Conflict-free repository operations
```

System guarantees:

```
credential isolation
deterministic authentication
scalable multi-account workflow
```

---

✅ **Multi-Account GitHub environment successfully stabilized**


