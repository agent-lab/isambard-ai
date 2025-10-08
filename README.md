# Isambard-AI Setup Guide <img src="images/brics.png" alt="BRICS Logo" width="50" height="50">

This document provides step-by-step instructions to set up and use **Isambard-AI**.

---

## 📑 Table of Contents
- [🚀 First Things First](#-first-things-first)  
- [💻 Connect via Terminal](#-connect-via-terminal)  
  - [1. Install Clifton](#1-install-clifton)  
  - [2. Generate an SSH Key](#2-generate-an-ssh-key)  
  - [3. Authenticate with Clifton](#3-authenticate-with-clifton)  
  - [4. Configure & Connect](#4-configure--connect)  
- [🖥️ Connect VS Code via Remote Tunnel](#️-connect-vs-code-via-remote-tunnel)  
  - [1. Install VS Code CLI on Login Node](#1-install-vs-code-cli-on-login-node)  
  - [2. Submit a Jobscript](#2-submit-a-jobscript)  
  - [3. Connect via GitHub](#3-connect-via-github)  
- [🛠️ Final Setup Steps](#️-final-setup-steps)

---

## 🚀 First Things First

1. Ask your supervisor for access to the **ISAMBARD-AI Supercluster**.  
2. Your supervisor will send an invitation to your **Manchester student email**. Accept the invitation.  
3. Go to the [Isambard Portal](https://portal.isambard.ac.uk/).  
4. When prompted to choose an access request type, select:  
   **University Login (MyAccessID)** → login with your Manchester email and password.  
5. Once access is granted to a project, you should see something like this:  
   ![Login Page](/images/login_portal.png)

---

## 💻 Connect via Terminal

The [BriCS facilities](https://docs.isambard.ac.uk/specs/) provide a command-line tool called [Clifton](https://github.com/isambard-sc/clifton).  
Clifton is used to obtain SSH certificates and configure your SSH client. Certificates are valid for **12 hours**.

### 1. Install Clifton

**Apple Silicon (macOS ARM64):**
```bash
curl -L https://github.com/isambard-sc/clifton/releases/latest/download/clifton-macos-aarch64 -o clifton
chmod u+x clifton
sudo mv clifton /usr/local/bin/
```

**Linux (including WSL):**
```bash
curl -L https://github.com/isambard-sc/clifton/releases/latest/download/clifton-linux-musl-x86_64 -o clifton
chmod u+x clifton
mkdir -p ~/.local/bin
mv clifton ~/.local/bin/
```

---

### 2. Generate an SSH Key
Follow [GitHub’s documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

> **Note:** Only modern SSH keys are supported (e.g. RSA ≥ 3072 bits, Ed25519).  
> Keys must include:  
> ```
> -----BEGIN OPENSSH PRIVATE KEY-----
> ```

---

### 3. Authenticate with Clifton
```bash
clifton auth --identity /path/to/ssh_key
```

- This will redirect you to a browser → login using **University Login (MyAccessID)**.
- After successful authentication, you’ll see something like:

```
Successfully authenticated as YOUR_EMAIL (SHORT_NAME) and downloaded SSH certificate for projects:
 - PROJECT_NAME

Certificate file written to ~/.ssh/id_ed25519-cert.pub
Certificate valid for 11 hours and 59 minutes.
You may now want to run `clifton ssh-config write` to configure your SSH config aliases.
```

---

### 4. Configure & Connect
1. Write the SSH key to your local config:
   ```bash
   clifton ssh-config write
   ```
2. Connect to the login node:
   ```bash
   ssh <PROJECT_ID>.aip2.isambard
   ```

---

## 🖥️ Connect VS Code via Remote Tunnel

### 1. Install VS Code CLI on Login Node
```bash
curl --location --output vscode_cli.tar.gz "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64"
mkdir -p ~/opt/vscode_cli
tar -C ~/opt/vscode_cli --extract --verbose --file vscode_cli.tar.gz
```

---

### 2. Submit a Jobscript

Create a file named **`vscode_code_tunnel.sh`**:
```bash
#!/bin/bash
#SBATCH --job-name=code_tunnel
#SBATCH --gpus=1            # Allocates 72 CPU cores + 115GB memory
#SBATCH --time=1:00:00
#SBATCH --output=code_tunnel_%j.out

module load brics/nano
# Start named VS Code tunnel for remote connection to compute node
~/opt/vscode_cli/code tunnel --name "i-ai_compute"
```

Run it:
```bash
sbatch vscode_code_tunnel.sh
```

---

### 3. Connect via GitHub

1. Open [GitHub Device Login](https://github.com/login/device).  
2. For the first run, you’ll be prompted for an **8-digit code**, which can be found in the jobscript output file, e.g. `code_tunnel_1182526.out`.  
3. Open VS Code → bottom-left corner → **Connect to Tunnel** → choose **GitHub**.  

> ✅ For subsequent connections:  
Just resubmit the jobscript and directly connect via **“Connect to Tunnel”** in VS Code.

> **Note:** If you update VS Code locally, redo **Step 1** (Install VS Code CLI on Login Node) to avoid version mismatch errors.

---

## 🛠️ Final Setup Steps

1. Install **Miniforge** to enable PyTorch on GPU:  
   [Guide: Installing Python Environment](https://docs.isambard.ac.uk/user-documentation/guides/python/#conda-installing-and-using-miniforge).
   
2. Follow this tutorial for setting up a **Distributed Training Environment in PyTorch**:  
   [Distributed Training Guide](https://docs.isambard.ac.uk/user-documentation/tutorials/distributed-training/).

3. Familiarize yourself with the **SLURM Workload Manager** used by Isambard:  
   [SLURM Documentation](https://docs.isambard.ac.uk/user-documentation/guides/slurm/).  
   > (It’s very similar to the CSF SLURM system.)

---

Happy Coding!