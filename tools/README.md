# 🛠️ Tools & Setup

> Build your offensive and defensive security environment.

---

## 🖥️ Lab Environment Setup

A dedicated VM (or bare-metal install) keeps your host system clean and lets you run potentially unsafe binaries safely. For malware analysis, use a fully isolated VM with no host network access.

---

### Recommended Operating Systems

| OS                                                    | Pros                                                        | Cons                                                  | Download       |
| ----------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------- | -------------- |
| [Kali Linux](https://www.kali.org)                    | 600+ preinstalled tools, huge community, familiar           | Heavy RAM use, bloated for non-security tasks         | ISO / VM image |
| [Parrot OS](https://parrotsec.org)                    | Lighter than Kali, good OPSEC tools included                | Smaller community, fewer preinstalled tools than Kali | ISO / VM image |
| [BlackArch](https://www.blackarch.org/downloads.html) | 3000+ tools in repo, Arch rolling-release                   | Complex setup, no hand-holding, Arch knowledge needed | ISO / VM image |
| [Garuda Linux](https://garudalinux.org/)              | Beautiful defaults, performance-tuned, BlackArch compitible | Gaming-focused origins, opinionated defaults          | ISO            |
| [Arch Linux](https://archlinux.org/download/)         | Full control, minimal base, rolling-release                 | Manual setup, steep learning curve                    | ISO / VM image |

---

### Virtual Machines (Recommended for Beginners)

Running your lab inside a VM keeps your host system intact. If something breaks or gets infected, you just delete or restore the VM — your real system stays clean.

#### VM Managers

| Software                                                                       | Host OS               | Pros                                          | Cons                                                   |
| ------------------------------------------------------------------------------ | --------------------- | --------------------------------------------- | ------------------------------------------------------ |
| [VirtualBox](https://www.virtualbox.org/)                                      | Windows, macOS, Linux | Free, open-source, snapshots, cross-platform  | Slower 3D/GPU, needs Guest Additions for full features |
| [VMware Workstation Pro](https://www.vmware.com/products/workstation-pro.html) | Windows, Linux        | Fast, polished UI, great hardware support     | Was paid (now free for personal use), closed-source    |
| [VMware Fusion](https://www.vmware.com/products/fusion.html)                   | macOS                 | Native macOS feel, Apple Silicon support      | Was paid (now free for personal use), macOS only       |
| [Parallels Desktop](https://www.parallels.com/)                                | macOS                 | Best Apple Silicon performance, seamless mode | Paid subscription, macOS only, closed-source           |
| [GNOME Boxes](https://apps.gnome.org/Boxes/)                                   | Linux                 | Dead simple setup, GNOME-integrated           | Limited config options, basic networking               |
| [Virt-Manager / KVM](https://virt-manager.org/)                                | Linux                 | Near-native performance, production-grade     | Complex setup, Linux host only                         |

> **Recommendation:** If you're new, start with **VirtualBox** — it's free, works on any OS, and has plenty of tutorials. On Linux, **Virt-Manager + KVM** is faster once you're comfortable.

---

### VirtualBox Setup

**Install VirtualBox:**

- Debian / Ubuntu

```bash
sudo apt update && sudo apt install -y virtualbox
```

- Arch / Garuda / BlackArch

```bash
sudo pacman -S virtualbox virtualbox-host-modules-arch
```

- macOS (Homebrew)

```bash
brew install --cask virtualbox
```

- Windows — download the installer from: https://www.virtualbox.org/wiki/Downloads

**Install VirtualBox Guest Additions (inside the VM, for clipboard & shared folders):**

```bash
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
# Then in VirtualBox menu: Devices → Insert Guest Additions CD Image → run autorun.sh
```

**Recommended VM settings for a security lab:**

| Setting                 | Recommended value                |
| ----------------------- | -------------------------------- |
| RAM                     | ≥ 4 GB (8 GB for heavy tools)    |
| CPU cores               | 2–4                              |
| Disk                    | ≥ 60 GB (dynamically allocated)  |
| Network adapter 1       | NAT (internet access)            |
| Network adapter 2       | Host-only (isolated lab traffic) |
| Clipboard / Drag & Drop | Bidirectional                    |

**Snapshot workflow — save your progress:**

```bash
# From the host terminal:
VBoxManage snapshot "Kali-Lab" take "clean-base" --description "Fresh install + updates"

# Restore if the VM breaks:
VBoxManage snapshot "Kali-Lab" restore "clean-base"
```

---

### Bare-Metal Setup

If you're installing directly on hardware, the steps are the same — skip VirtualBox and just boot from the ISO.

1. Download the ISO for your chosen OS.
2. Flash to USB: `sudo dd if=kali-linux.iso of=/dev/sdX bs=4M status=progress && sync`
   (or use [Balena Etcher](https://etcher.balena.io/))
3. Boot from USB and follow the installer.
4. On first boot, update the system:

```bash
sudo apt update && sudo apt full-upgrade -y && sudo reboot
```

---

### Starter Tools

Install these first on any fresh security lab machine.

#### System essentials

```bash
sudo apt install -y \
  git curl wget vim neovim tmux \
  build-essential gcc g++ make cmake \
  python3 python3-pip python3-venv \
  net-tools dnsutils ncat socat \
  unzip p7zip-full jq tree htop \
  golang-go ruby-full default-jdk
```

#### VS Code

```bash
# Download and install the .deb package
wget -qO /tmp/vscode.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64"
sudo dpkg -i /tmp/vscode.deb
sudo apt install -f -y   # fix any dependency issues

# Useful extensions for security work
code --install-extension ms-vscode.hexeditor          # hex viewer
code --install-extension ms-python.python             # Python support
code --install-extension redhat.vscode-yaml           # YAML support
code --install-extension streetsidesoftware.code-spell-checker
```

#### Git configuration

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "vim"
git config --global init.defaultBranch main

# Generate an SSH key for GitHub / GitLab
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub   # paste this into GitHub → Settings → SSH Keys
```

#### Python environment

```bash
# Upgrade pip and install common security libs
pip install --upgrade pip
pip install pwntools requests beautifulsoup4 scapy impacket
```

#### Go environment

```bash
# Ensure Go bin directory is on PATH (add to ~/.zshrc or ~/.bashrc)
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.zshrc
source ~/.zshrc
```

---

## 🔴 Red Team Tools

### Reconnaissance & OSINT

| Tool           | Purpose                           | Install                                                                    |
| -------------- | --------------------------------- | -------------------------------------------------------------------------- |
| `nmap`         | Network port scanner              | `apt install nmap`                                                         |
| `masscan`      | Fast mass port scanner            | `apt install masscan`                                                      |
| `subfinder`    | Passive subdomain enumeration     | `go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` |
| `amass`        | Subdomain enumeration and mapping | `apt install amass`                                                        |
| `theHarvester` | Email/domain OSINT                | `apt install theharvester`                                                 |
| `sherlock`     | Username search across platforms  | `pip install sherlock-project`                                             |

### Web Exploitation

| Tool           | Purpose                              | Install                                                              |
| -------------- | ------------------------------------ | -------------------------------------------------------------------- |
| Burp Suite Pro | HTTP proxy, scanner, intruder        | [portswigger.net](https://portswigger.net/burp/pro)                  |
| `ffuf`         | Web fuzzing                          | `go install github.com/ffuf/ffuf/v2@latest`                          |
| `sqlmap`       | SQL injection automation             | `apt install sqlmap`                                                 |
| `nuclei`       | Template-based vulnerability scanner | `go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest` |
| `nikto`        | Web vulnerability scanner            | `apt install nikto`                                                  |

### Active Directory & Network Attacks

| Tool                | Purpose                                         | Install                                              |
| ------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| `impacket`          | AD/SMB attack suite (psexec, secretsdump, etc.) | `pip install impacket`                               |
| `bloodhound-python` | BloodHound data collection                      | `pip install bloodhound`                             |
| BloodHound          | AD attack path visualisation                    | [GitHub](https://github.com/BloodHoundAD/BloodHound) |
| `evil-winrm`        | WinRM shell                                     | `gem install evil-winrm`                             |
| `responder`         | LLMNR/NBT-NS poisoning                          | `apt install responder`                              |
| `crackmapexec`      | AD Swiss-army knife                             | `apt install crackmapexec`                           |
| `kerbrute`          | Kerberos user enumeration and bruteforce        | [GitHub](https://github.com/ropnop/kerbrute)         |

### Post-Exploitation & C2

| Tool                                          | Purpose                                      | Install                                          |
| --------------------------------------------- | -------------------------------------------- | ------------------------------------------------ |
| Metasploit                                    | Exploitation and post-exploitation framework | `apt install metasploit-framework`               |
| [Sliver](https://github.com/BishopFox/sliver) | Open-source C2 framework                     | GitHub releases                                  |
| `msfvenom`                                    | Payload generation                           | bundled with Metasploit                          |
| `chisel`                                      | TCP/UDP tunnel over HTTP                     | `go install github.com/jpillora/chisel@latest`   |
| `ligolo-ng`                                   | Tunneling for pivoting                       | [GitHub](https://github.com/nicocha30/ligolo-ng) |

### Privilege Escalation

| Tool       | Purpose                                  | Install                                           |
| ---------- | ---------------------------------------- | ------------------------------------------------- |
| `linpeas`  | Linux privilege escalation enumeration   | [GitHub](https://github.com/carlospolop/PEASS-ng) |
| `winpeas`  | Windows privilege escalation enumeration | [GitHub](https://github.com/carlospolop/PEASS-ng) |
| `mimikatz` | Windows credential dumping               | [GitHub](https://github.com/gentilkiwi/mimikatz)  |
| `Rubeus`   | Kerberos manipulation                    | [GitHub](https://github.com/GhostPack/Rubeus)     |

---

## 🔵 Blue Team Tools

### SIEM & Log Analysis

| Tool                                                      | Purpose                                         | Install                                           |
| --------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------- |
| [Elastic Stack (ELK)](https://www.elastic.co)             | Log ingestion, search, and visualisation        | Docker / APT                                      |
| [Splunk Free](https://www.splunk.com/en_us/download.html) | SIEM with 500MB/day free tier                   | Installer                                         |
| [Wazuh](https://wazuh.com)                                | Open-source SIEM + HIDS                         | [Install guide](https://documentation.wazuh.com/) |
| `sigma`                                                   | Convert generic detection rules to SIEM queries | `pip install sigma-cli`                           |

### Endpoint Detection & Forensics

| Tool                                                                      | Purpose                                            | Install                                 |
| ------------------------------------------------------------------------- | -------------------------------------------------- | --------------------------------------- |
| [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) | Detailed Windows event logging                     | Windows                                 |
| [Velociraptor](https://github.com/Velocidex/velociraptor)                 | DFIR and endpoint monitoring                       | GitHub releases                         |
| [Volatility 3](https://github.com/volatilityfoundation/volatility3)       | Memory forensics                                   | `pip install volatility3`               |
| [Eric Zimmerman Tools](https://ericzimmerman.github.io)                   | Windows forensics utilities (PECmd, MFTECmd, etc.) | Windows                                 |
| `autopsy`                                                                 | Digital forensics GUI                              | [autopsy.com](https://www.autopsy.com/) |

### Network Monitoring & Analysis

| Tool                            | Purpose                              | Install                 |
| ------------------------------- | ------------------------------------ | ----------------------- |
| [Zeek (Bro)](https://zeek.org)  | Network traffic analysis and logging | `apt install zeek`      |
| [Suricata](https://suricata.io) | Network IDS/IPS                      | `apt install suricata`  |
| Wireshark / `tshark`            | PCAP analysis                        | `apt install wireshark` |
| `zeek-cut`                      | Zeek log field extractor             | bundled with Zeek       |

### Malware Analysis

| Tool                                               | Purpose                                 | Install                                 |
| -------------------------------------------------- | --------------------------------------- | --------------------------------------- |
| [REMnux](https://remnux.org)                       | Linux malware analysis distro           | VM download                             |
| [FlareVM](https://github.com/mandiant/flare-vm)    | Windows malware analysis toolkit        | Windows setup script                    |
| `FLOSS`                                            | Extract obfuscated strings from malware | `pip install flare-floss`               |
| `pestudio`                                         | PE file static analysis                 | [winitor.com](https://www.winitor.com/) |
| `YARA`                                             | Pattern-matching for malware detection  | `apt install yara`                      |
| [Any.run](https://app.any.run)                     | Interactive online sandbox              | Browser                                 |
| [Hybrid-Analysis](https://www.hybrid-analysis.com) | Automated online sandbox                | Browser                                 |

### Threat Intelligence

| Tool                                 | Purpose                       | Install                                                 |
| ------------------------------------ | ----------------------------- | ------------------------------------------------------- |
| [MISP](https://www.misp-project.org) | Threat intel sharing platform | [Install guide](https://www.misp-project.org/download/) |
| [OpenCTI](https://www.opencti.io)    | Threat intel management       | Docker                                                  |

---

## General Security Tools by Category

### General

| Tool              | Purpose                                  | Install                                                       |
| ----------------- | ---------------------------------------- | ------------------------------------------------------------- |
| `file`            | Identify file type                       | built-in                                                      |
| `strings`         | Extract printable strings from binary    | built-in                                                      |
| `xxd` / `hexdump` | Hex viewer                               | built-in                                                      |
| `binwalk`         | Firmware/archive analysis and extraction | `pip install binwalk`                                         |
| `exiftool`        | Read/write file metadata                 | `apt install libimage-exiftool-perl`                          |
| `foremost`        | File carving                             | `apt install foremost`                                        |
| `7z` / `unzip`    | Archive extraction                       | `apt install p7zip-full`                                      |
| CyberChef         | Browser-based encode/decode/transform    | [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef/) |

### Web

| Tool            | Purpose                            | Install                                         |
| --------------- | ---------------------------------- | ----------------------------------------------- |
| Burp Suite      | HTTP proxy & scanner               | [portswigger.net](https://portswigger.net/burp) |
| `curl` / `wget` | HTTP requests from CLI             | built-in                                        |
| `ffuf`          | Web fuzzing (dirs, params, vhosts) | `go install github.com/ffuf/ffuf/v2@latest`     |
| `gobuster`      | Directory/DNS brute-force          | `apt install gobuster`                          |
| `sqlmap`        | Automated SQL injection            | `apt install sqlmap`                            |
| `nikto`         | Web vulnerability scanner          | `apt install nikto`                             |
| Wappalyzer      | Technology fingerprinting          | browser extension                               |

### Cryptography

| Tool         | Purpose                                 | Install                                            |
| ------------ | --------------------------------------- | -------------------------------------------------- |
| `openssl`    | Swiss-army crypto tool                  | built-in                                           |
| `hashcat`    | GPU password cracking                   | `apt install hashcat`                              |
| `john`       | CPU password cracking (John the Ripper) | `apt install john`                                 |
| SageMath     | Math-heavy crypto scripting             | [sagemath.org](https://www.sagemath.org/)          |
| CyberChef    | Quick encode/decode                     | browser                                            |
| `RsaCtfTool` | RSA attack automation                   | [GitHub](https://github.com/RsaCtfTool/RsaCtfTool) |

### Forensics

| Tool                 | Purpose                           | Install                                                       |
| -------------------- | --------------------------------- | ------------------------------------------------------------- |
| Wireshark / `tshark` | PCAP analysis                     | `apt install wireshark`                                       |
| Volatility 3         | Memory forensics                  | [GitHub](https://github.com/volatilityfoundation/volatility3) |
| Autopsy              | Digital forensics GUI             | [autopsy.com](https://www.autopsy.com/)                       |
| `scalpel`            | File carving                      | `apt install scalpel`                                         |
| `NetworkMiner`       | Passive network sniffer/forensics | [netresec.com](https://www.netresec.com/)                     |

### Reverse Engineering

| Tool                | Purpose                        | Install                                                        |
| ------------------- | ------------------------------ | -------------------------------------------------------------- |
| Ghidra              | NSA decompiler (free)          | [ghidra-sre.org](https://ghidra-sre.org/)                      |
| IDA Free            | Industry-standard disassembler | [hex-rays.com](https://hex-rays.com/ida-free/)                 |
| Binary Ninja        | Modern disassembler/decompiler | [binary.ninja](https://binary.ninja/)                          |
| `gdb` + `pwndbg`    | Debugger with exploit helpers  | `apt install gdb` + [pwndbg](https://github.com/pwndbg/pwndbg) |
| `strace` / `ltrace` | Syscall / library call tracing | built-in                                                       |
| `objdump`           | Object file disassembly        | built-in                                                       |
| `radare2`           | Reverse engineering framework  | `apt install radare2`                                          |

### PWN / Binary Exploitation

| Tool               | Purpose                                  | Install                                                  |
| ------------------ | ---------------------------------------- | -------------------------------------------------------- |
| `pwntools`         | CTF exploit development framework        | `pip install pwntools`                                   |
| `ROPgadget`        | ROP chain finder                         | `pip install ROPgadget`                                  |
| `checksec`         | Check binary security flags              | `pip install checksec.sh`                                |
| `one_gadget`       | Find one-shot RCE gadgets in libc        | `gem install one_gadget`                                 |
| `patchelf`         | Modify ELF binary interpreter/RPATH      | `apt install patchelf`                                   |
| `glibc-all-in-one` | Multiple libc versions for local testing | [GitHub](https://github.com/matrix1001/glibc-all-in-one) |

### Steganography

| Tool        | Purpose                            | Install                                                                       |
| ----------- | ---------------------------------- | ----------------------------------------------------------------------------- |
| `steghide`  | Embed/extract data in images/audio | `apt install steghide`                                                        |
| `stegsolve` | Image stego analysis GUI           | [GitHub](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve) |
| `zsteg`     | PNG/BMP steganography detector     | `gem install zsteg`                                                           |
| `outguess`  | Stego in JPEG                      | `apt install outguess`                                                        |
| `audacity`  | Audio spectrogram analysis         | `apt install audacity`                                                        |
| `sox`       | Audio processing CLI               | `apt install sox`                                                             |

---

## Quick Setup Script

```bash
# Update and install base tools
sudo apt update && sudo apt install -y \
  binwalk exiftool foremost p7zip-full \
  curl wget gobuster sqlmap nikto nmap masscan amass \
  wireshark tshark gdb strace ltrace \
  radare2 steghide outguess audacity \
  john hashcat patchelf suricata zeek \
  yara responder

# Python tools
pip install pwntools ROPgadget impacket bloodhound sigma-cli volatility3 flare-floss

# Ruby tools
gem install one_gadget zsteg evil-winrm

# Go tools
go install github.com/ffuf/ffuf/v2@latest
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install github.com/jpillora/chisel@latest
```
