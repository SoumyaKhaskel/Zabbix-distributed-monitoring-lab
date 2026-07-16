## Challenges Faced & Solutions — Zabbix Lab Project

---

### 1. Zabbix Server cannot run natively on macOS
**Challenge:** Zabbix Server is a Linux binary. No native macOS installer exists.
**Solution:** Deployed the full stack (Zabbix Server + MySQL + Frontend) using Docker Compose, providing a Linux environment on macOS without dual booting or a separate machine.

---

### 2. Docker containers not auto-starting after Mac reboot
**Challenge:** After restarting the MacBook, all three containers were stopped and Zabbix was completely down.
**Solution:** Enabled Docker Desktop auto-start on login via settings. Containers already had `restart: unless-stopped` policy so they self-recover once Docker Desktop is running.

---

### 3. Homebrew installed old Zabbix Agent instead of Agent 2
**Challenge:** Running `brew install zabbix` installed the classic `zabbix_agentd` binary instead of the modern Agent 2.
**Solution:** Used the classic agent instead — it covers all required metrics for this lab. Located the exact binary path under Homebrew Cellar and used it directly for all subsequent commands.

---

### 4. Agent refused to run as root
**Challenge:** Running `sudo zabbix_agentd` returned `cannot run as root` error immediately.
**Solution:** Created a dedicated `zabbix` system user on macOS using `dscl` commands, allowing the agent to drop privileges safely on startup — matching production security standards.

---

### 5. Zabbix Server inside Docker couldn't reach Mac agent via 127.0.0.1
**Challenge:** Server and agent were on the same physical machine but Docker's network namespace means `127.0.0.1` inside a container points to the container itself, not the Mac host.
**Solution:** Used Docker Desktop's built-in `host.docker.internal` DNS name which always resolves to the Mac host from inside any container. Changed the host interface in Zabbix frontend from IP to DNS mode.

---

### 6. Zabbix Agent not surviving Mac reboots
**Challenge:** Every time the Mac restarted the agent process was gone and had to be started manually.
**Solution:** Created a macOS LaunchDaemon plist file at `/Library/LaunchDaemons/com.zabbix.agentd.plist` with `RunAtLoad=true` and `KeepAlive=true` for permanent auto-start. Also created shell aliases `zabbix-start` and `zabbix-stop` for quick manual control.

---

### 7. Windows Desktop blocking all connections by default
**Challenge:** Windows 11 Desktop at `192.168.1.3` showed 100% packet loss on ping — completely unreachable from Mac.
**Solution:** Added Windows Defender Firewall inbound rules via PowerShell (`netsh advfirewall`) to allow ICMPv4 (ping) and TCP port 10050 (Zabbix Agent).

---

### 8. Windows Desktop IP conflict with router
**Challenge:** Running `ipconfig` on Windows returned `192.168.1.1` which is the Airtel router's gateway IP — causing confusion about the actual device IP.
**Solution:** Used `ipconfig | findstr "IPv4"` and cross-referenced with router ARP table to correctly identify the desktop's actual LAN IP as `192.168.1.3` (later changed to `192.168.1.4`).

---

### 9. Remote PCs on different ISPs unreachable
**Challenge:** Subhro (`103.240.98.16`) and Randojoi (`49.37.37.38`) are in different cities on different ISPs. Private IPs are not routable across the internet. No access to Airtel router to configure port forwarding.
**Solution:** Evaluated multiple approaches — Ngrok (required card), Cloudflare Tunnel (TCP protocol incompatible with Zabbix binary protocol), Active Agent Mode (needs public server IP). Final solution: Tailscale mesh VPN providing stable `100.x.x.x` IPs across all devices regardless of ISP or NAT.

---

### 10. TryCloudflare tunnel incompatible with Zabbix protocol
**Challenge:** Cloudflare's free tunnel wrapped TCP traffic in HTTPS. Zabbix uses its own binary protocol over raw TCP which Cloudflare's HTTPS wrapper breaks silently.
**Solution:** Abandoned Cloudflare tunnel approach entirely and switched to Tailscale which provides true Layer 3 raw TCP connectivity, fully compatible with Zabbix's native protocol.

---

### 11. Remote PCs joined separate Tailscale networks
**Challenge:** Randojoi logged into Tailscale with his own Google account (`ranajayexe@gmail.com`) instead of yours. Subhro did the same. Devices on different Tailscale accounts cannot communicate — they are on completely separate tailnets.
**Solution (pending):** Use Tailscale admin console invite link to add remote devices to your tailnet without sharing passwords — resuming next session.

---

## Summary table for resume/documentation

| # | Challenge | Root Cause | Solution |
|---|---|---|---|
| 1 | Zabbix won't run on macOS | Linux-only binary | Docker Compose |
| 2 | Containers stop on reboot | Docker not auto-starting | Docker Desktop login setting |
| 3 | Wrong agent version installed | Homebrew formula limitation | Used classic agent |
| 4 | Agent refused root execution | Security restriction | Dedicated zabbix system user |
| 5 | Agent unreachable from Docker | Network namespace isolation | host.docker.internal |
| 6 | Agent lost on reboot | No persistence configured | LaunchDaemon plist |
| 7 | Windows blocking connections | Default firewall rules | PowerShell firewall rules |
| 8 | IP address confusion | Router gateway vs device IP | ipconfig filtering |
| 9 | Remote PCs unreachable | Different ISP, no router access | Tailscale mesh VPN |
| 10 | Cloudflare tunnel incompatible | HTTPS wrapping breaks TCP | Switched to Tailscale |
| 11 | Tailscale separate networks | Wrong account login | Admin console invite (pending) |

---

Every single one of these is a real-world problem you will face again in production environments. The fact that you hit them, diagnosed them, and solved them is exactly what makes this project genuinely valuable on your resume — not just "I installed Zabbix." 🔥
