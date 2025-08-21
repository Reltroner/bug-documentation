# 🛠️ Case 1 — Generate SSH Key & Configure GitHub

This document describes the process of generating a dedicated SSH key for GitHub authentication and configuring it for secure access.

---

## 1. Generate SSH Key

Run the following command to create a new **RSA 4096-bit** SSH key pair:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_github -N ""
````

**Result:**

* Private key saved to: `~/.ssh/id_rsa_github`
* Public key saved to: `~/.ssh/id_rsa_github.pub`
* Key fingerprint and randomart image are generated for identification.

---

## 2. Add Public Key to Authorized Keys (server-side)

Append the public key to your authorized keys:

```bash
cat ~/.ssh/id_rsa_github.pub >> ~/.ssh/authorized_keys
```

---

## 3. Configure SSH for GitHub

Create a new SSH config file:

```bash
cat > ~/.ssh/config <<'EOF'
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_github
  IdentitiesOnly yes
EOF
```

Secure the permissions:

```bash
chmod 600 ~/.ssh/config
```

---

## 4. Verify the Public Key

Check the generated key:

```bash
cat ~/.ssh/id_rsa_github.pub
```

---

## 5. Test GitHub Authentication

Test your SSH connection to GitHub:

```bash
ssh -T git@github.com
```

**Expected Output:**

```
Hi Reltroner/reltroner-hr-app! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## ✅ Outcome

* SSH key pair successfully created and configured.
* GitHub authentication verified via SSH.
* Ready to use `git clone`, `git pull`, and `git push` securely with the configured key.
