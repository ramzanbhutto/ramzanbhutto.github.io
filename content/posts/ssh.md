---
title: "Secure Shell (SSH)"
date: "2026-02-09"
draft: false
showToc: true
Author: "Muhammad Ramzan"
tags: [ssh", "linux"]
---

## Overview
SSH (Secure Shell) allows you to securely connect to a remote computer over the internet. It encrypts your connection, letting you execute commands safely on another machine without physical access. SSH enables remote management, secure file transfers, and automation. It encrypts communication to prevent unauthorized access.

---

## Requirements
- Linux distribution (server and client)
- Sudo privileges on the server
- Internet connection 

---

## Setting Up an SSH Server

### Step 1: Installing SSH

```sh
sudo pacman -Sy openssh
```

### Step 2: Enable, Start and Verify SSH Daemon

```sh
sudo systemctl enable sshd
sudo systemctl start sshd
sudo systemctl status sshd
```

### Step 3: Configuring the Firewall
If you have a firewall enabled, you need to allow SSH connections through it.

```sh
sudo ufw enable  # If not already enabled, check using 'sudo ufw status'
sudo ufw allow ssh
sudo ufw reload
```
> TCP port `22` is used by default for SSH connections.

## Connecting from a Client

### Step 1: Install SSH

```sh
sudo pacman -Sy openssh
```

### Step 2: Find Server's IP address
On the server, run the following command to find its IP address:

```sh
ip addr show
```
> Look for the `inet` entry under the network interface you are using (e.g., `eth0`, `wlan0`). It will look something like this: *192.168.1.10*

### Step 3: Connect to Server

```sh
ssh username@server_ip
```
> The first connection may show a warning about server authenticity. This is normal.
---

## Changing SSH Port

### Step 1: Editing the configuration
By default SSH uses the port `22` for connections. You can change it.

```sh
sudo nvim /etc/ssh/sshd_config
```
> You need `sudo` permissions for this.

1. Find the following line: ` #Port 22`
2. Uncomment by removing the `#` and changing the port to your desired port. For example, `Port 4235`
3. Save and exit the file.

### Step 2: Restarting the SSH Service

```sh
sudo systemctl restart sshd
```

### Step 3: Update Firewall Rules
You need to allow the new port through the firewall. For example, if you changed it to `4235`, run:

```sh
sudo ufw allow 4235/tcp
sudo ufw reload
```

### Step 4: Check the firewall status
Check if the newly assigned port has been allowed:

```sh
sudo ufw status
```

### Step 5: Connecting using new port
If port `4235` was assigned, connect using:

```sh
ssh -p 4235 username@server_ip
```

---

## Optional: Key-Based Authentication
Key-based authentication is a more secure way to connect to your SSH server compared to password authentication. It uses a pair of cryptographic keys: a public key and a private key.

### Step 1: Generating the Key

```sh
ssh-keygen
```
> Press `Enter` to accept the default file location and name. You can also set a passphrase for added security, but it's optional.

This will create two files in your `~/.ssh/` directory:
```sh
~/.ssh/id_ed25519        ← private key
~/.ssh/id_ed25519.pub    ← public key
```

### Step 2: Copy the Public Key to the Server

```sh
ssh-copy-id username@server_ip
```

After entering your password, SSH login will use your key instead of a password.

---
