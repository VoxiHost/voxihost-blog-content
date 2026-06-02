---
image: /assets/images/blog/en/transfer-files-vps-sftp-filezilla/og-image.png
title: How to Transfer Files to Your VPS using SFTP & FileZilla
description: A complete beginner's guide to securely transferring files from your PC to your Linux VPS using SFTP, FileZilla, and SSH Keys.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: transfer-files-vps-sftp-filezilla
category: Tutorials
tags:
  - sftp
  - filezilla
  - ftp
  - ssh
  - linux
  - vps
  - file transfer
  - ubuntu
  - debian
howto:
  name: How to Connect to a VPS using SFTP and FileZilla
  totalTime: PT5M
  yield: A secure, visual file transfer connection capable of dragging and dropping files directly to your Linux server
  tool:
    - A Linux VPS or dedicated server
    - FileZilla FTP Client installed on your desktop
    - Your server IP and SSH Key (or password)
  steps:
    - name: Download FileZilla
      text: Download and install the free FileZilla client from their official website.
      url: step-1-download-filezilla
    - name: Open the Site Manager
      text: Click the Site Manager icon in the top left corner to create a permanent, saved connection.
      url: step-2-configure-the-site-manager
    - name: Select the SFTP Protocol
      text: Change the Protocol dropdown from standard 'FTP' to 'SFTP - SSH File Transfer Protocol'.
      url: step-3-select-the-sftp-protocol
    - name: Enter your SSH Key or Password
      text: Change Logon Type to 'Key file', browse for your private SSH key, and enter your server IP and standard SSH port (22).
      url: step-4-add-your-credentials-or-ssh-key
    - name: Connect and transfer
      text: Click Connect, accept the host key warning, and drag-and-drop your files between your PC and the server.
      url: step-5-connect-and-transfer
faq:
  - question: "What is the difference between FTP and SFTP?"
    answer: "FTP (File Transfer Protocol) transfers data and credentials in plain text, making it insecure. SFTP (SSH File Transfer Protocol) runs inside a secure SSH tunnel, encrypting both authentication credentials and files in transit."
  - question: "Which port does SFTP use?"
    answer: "By default, SFTP uses port <code>22</code>, which is the standard SSH port. If you have changed your server's SSH port (e.g. to <code>2222</code>), you must specify this custom port in FileZilla's Port field."
  - question: "How do I fix permission denied errors when uploading files?"
    answer: "Permission denied errors occur when the SSH user you connected with does not own the target directory. You can fix this by running <code>sudo chown -R username:username /path/to/directory</code> on your server."
  - question: "How do I show hidden files (dotfiles like .htaccess) in FileZilla?"
    answer: "In FileZilla, go to the top menu, select <b>Server</b>, and check <b>Force showing hidden files</b> to display files starting with a dot."
  - question: "Can I use PuTTY private keys (.ppk) with FileZilla?"
    answer: "Yes. FileZilla fully supports PuTTY's <code>.ppk</code> format. When using a standard OpenSSH key, FileZilla will prompt to automatically convert it to <code>.ppk</code> format for Windows users."
status: published
locale: en
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

When you rent a brand new Linux VPS, you are usually greeted with a terrifying black terminal screen. 

While you can technically upload files directly through the command line using `scp` or `rsync`, dragging and dropping your website's folder using a visual interface is infinitely easier, especially for beginners.

The absolute best way to do this is using **SFTP** (Secure File Transfer Protocol) with a free desktop client called **FileZilla**. 

**Important Note:** Do not confuse standard FTP with SFTP. Standard FTP sends your passwords and files over the internet in plain, unencrypted text. SFTP routes all FTP commands through your server's secure SSH tunnel. Because of this, *you do not need to install an FTP server like vsftpd on your Linux VPS*. If you have an SSH connection, SFTP will work automatically!

## Step 1: Download FileZilla

{% image "/assets/images/blog/en/transfer-files-vps-sftp-filezilla/H1.png", "Downloading FileZilla Client installer from the official filezilla-project.org website for SFTP connection", "(max-width: 768px) 100vw, 800px" %}

If you haven't already, download the **FileZilla Client** (not the Server version) from the [official FileZilla website](https://filezilla-project.org/). It is completely free and available for Windows, macOS, and Linux.

## Step 2: Configure the Site Manager

{% image "/assets/images/blog/en/transfer-files-vps-sftp-filezilla/H2.png", "Opening the FileZilla Site Manager to create a new saved SFTP server connection", "(max-width: 768px) 100vw, 800px" %}

Do **not** use the "Quickconnect" bar at the very top of the application. It does not securely save your SSH keys or complex server settings. 

Instead, open the **Site Manager**. You can find it by clicking the very first icon on the top-left toolbar, or by navigating to `File > Site Manager` (or `CTRL + S` on Windows).

1. Click **New Site**.
2. Name the site something recognizable, like "My VPS Web Server".

## Step 3: Select the SFTP Protocol

{% image "/assets/images/blog/en/transfer-files-vps-sftp-filezilla/H3.png", "Selecting SFTP - SSH File Transfer Protocol from the FileZilla protocol dropdown instead of FTP", "(max-width: 768px) 100vw, 800px" %}

Look at the right side of the Site Manager window. Under the **Protocol** dropdown, it defaults to standard FTP. 

You **must** change this. Click the dropdown and select:
**`SFTP - SSH File Transfer Protocol`**

If you skip this step, FileZilla will constantly fail to connect to your server securely.

## Step 4: Add Your Credentials or SSH Key

{% image "/assets/images/blog/en/transfer-files-vps-sftp-filezilla/H4.png", "Adding VPS IP address, username, and SSH key file in FileZilla Site Manager for secure connection", "(max-width: 768px) 100vw, 800px" %}

Now you need to tell FileZilla where to go and how to log in.

1. **Host**: Enter your server's public IP address (e.g., `192.168.1.100`).
2. **Port**: Leave this **blank** unless you have manually changed your SSH port.
3. **Logon Type**: This is where most beginners get stuck. Change this to **`Key file`** if you use SSH keys, or **`Normal`** if you use a password.
4. **User**: Enter your username (usually **`root`** for a fresh server deployment).
5. **Key file**: Click **Browse** and select the private key file on your computer.

### If you use a Password:
While we highly discourage using passwords for server access (as they are vulnerable to brute-force attacks), if you must:
1. Set the Logon Type to **Normal**.
2. **User**: Type `root` (or whatever user account you created).
3. **Password**: Enter your standard SSH password.

### If you use SSH Keys (Recommended):
If you generated an [SSH key pair](/blog/secure-ssh-ubuntu-debian/) (like `id_rsa` or `id_ed25519`) to log into your server securely without a password:
1. Set the Logon Type to **Key file**.
2. **User**: Type your username (like `root` or `admin`).
3. Click the **Browse** button and locate the `private key` file on your desktop (not the one ending in `.pub`). 

*(If FileZilla asks for a password during the connection, it is asking for the passphrase you put on your private key, not the server's root password).*

## Step 5: Connect and Transfer

{% image "/assets/images/blog/en/transfer-files-vps-sftp-filezilla/H5.png", "FileZilla host key verification popup when connecting to a VPS for the first time via SFTP", "(max-width: 768px) 100vw, 800px" %}

Click the **Connect** button at the bottom of the window.

The very first time you connect to the server from this computer, a scary-looking window will pop up titled *"Unknown host key"*. This is a **standard security measure** preventing "Man in the Middle" attacks. 
Check the box that says **"Always trust this host..."** and click OK.

### The FileZilla Interface

{% image "/assets/images/blog/en/transfer-files-vps-sftp-filezilla/H6.png", "FileZilla split-pane interface showing local files on left and remote Linux VPS directory on right for drag-and-drop transfer", "(max-width: 768px) 100vw, 800px" %}

If your credentials are correct, you will successfully connect. You are now looking at two massive split windows:
- **Left Side**: Your **local computer** (your hard drive).
- **Right Side**: Your **Linux VPS** (the remote server).

To transfer files, simply **drag and drop** them from the left window to the right window.

A common place to upload web files is:
`/var/www/html/`

Navigate there in the right-side window, drag your `index.html` from your desktop (left side) to that folder, and your website is live!

If you don't have a server to practice on, **[Budget VPS](/budget-vps/)** plans from **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>** are a perfect, affordable playground to learn how to manage Linux without breaking the bank. You can deploy a clean instance in seconds and start transferring files immediately.