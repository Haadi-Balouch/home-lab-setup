# Lessons Learned

Honest notes on every real problem I hit while building this lab, how I solved
each one, and what I would do differently. Updated as the lab grows.

This file matters because security work is mostly problem-solving under pressure.
Documenting how I think through problems is as important as documenting the lab itself.

---

## Problem 1 — Metasploitable 2 Would Not Open in VirtualBox

### What happened
Metasploitable 2 is distributed as a VMware `.vmdk` disk image. When I tried to
open it directly in VirtualBox, it did not work the way I expected. VirtualBox
does not have a simple "open this VMware VM" button — the process is different
from how VMware handles it, and the documentation around this was not obvious.

### How I figured it out
Googled the error and found that the `.vmdk` file itself is usable by VirtualBox,
but the VMware `.vmx` configuration file is not. The correct approach in VirtualBox is:

1. Create a **new VM manually** — do not try to import the `.vmx`
2. When asked for a disk, choose **"Use an existing virtual hard disk file"**
3. Point it at the Metasploitable `.vmdk` file directly
4. Set OS type to **Other Linux (32-bit)** — this was the key setting I missed initially

Without setting the correct OS type, the VM either would not boot or threw errors
about incompatible hardware settings.

### What I learned
VMware and VirtualBox use different configuration formats. The disk (`.vmdk`) is
portable between them but the VM settings are not. Any pre-built VM downloaded
for VMware needs to be manually recreated in VirtualBox — the disk just gets
attached to a fresh VM shell configured with the right settings.

This is a common pattern: many security lab VMs online are built for VMware.
Knowing how to convert them to VirtualBox is a practical skill I now have.

---

## Problem 2 — Host-Only Network Did Not Exist

### What happened
After setting both VMs to Host-Only networking in VirtualBox, they still could
not communicate. Running `ping 192.168.56.102` from Kali returned nothing.
Checking `ip addr show` on Kali showed the interface existed but had no IP address.

The root cause was that VirtualBox had no Host-Only network configured at the
host level — the network itself did not exist yet.

### How I fixed it
- Opened VirtualBox → **File → Host Network Manager**
- Clicked **Create** — this generated `vboxnet0`
- Set the subnet to `192.168.56.0/24` and enabled the DHCP server
- Rebooted both VMs

After this, both VMs received IP addresses on the `192.168.56.x` range and
could ping each other successfully.

### What I learned
VirtualBox does not create a Host-Only network automatically. It is a one-time
manual setup that has to happen before any VM can use it. This step is missing
from most beginner tutorials, which is probably why it is one of the most
commonly Googled VirtualBox problems.

---

## Problem 3 — Kali Had No IP on the Host-Only Interface After Reboot

### What happened
Even after the Host-Only network was created and working, rebooting Kali caused
it to lose its IP on `eth1` — the Host-Only adapter. The interface existed but
showed no IP. Metasploitable became unreachable again after every reboot.

### How I fixed it
Ran this manually each time as a short-term fix:
```bash
sudo dhclient eth1
```

Then made it permanent by editing the network interfaces file:
```bash
sudo nano /etc/network/interfaces
```
Added:
```
auto eth1
iface eth1 inet dhcp
```

After this, `eth1` gets an IP automatically every time Kali boots.

### What I learned
Adding a network adapter in VirtualBox settings does not automatically configure
it at the Linux OS level. The interface needs to be told to come up on boot.
This is a Linux networking fundamental — the OS manages interfaces independently
of what the hypervisor provides.

---

## Problem 4 — Had to Reconfigure Network Settings From Scratch

### What happened
At one point I changed network adapter settings while VMs were in a broken state
and made things worse. Adapters were set to the wrong types, IPs were conflicting,
and Kali could no longer reach Metasploitable at all. Rather than debug a
complicated broken state, I reset both VMs' network configuration completely.

### How I fixed it
- Shut down both VMs
- Went into VirtualBox settings for each VM → Network → removed and re-added adapters cleanly
- Set Kali: Adapter 1 = NAT, Adapter 2 = Host-Only (vboxnet0)
- Set Metasploitable: Adapter 1 = Host-Only (vboxnet0) only
- Booted both, ran `sudo dhclient eth1` on Kali, verified IPs with `ip addr show`
- Tested with `ping 192.168.56.102` — success

Total time to reconfigure from scratch: about 20 minutes.

### What I learned
Sometimes the fastest fix is a clean reset rather than trying to debug a
complicated broken state. Knowing the correct configuration clearly means
rebuilding it takes minutes. This is why documentation matters — I had my
correct settings written down and could restore them quickly.

---

## Problem 5 — Lost Lab Progress Due to No Snapshots

### What happened
Early in the lab setup I made several configuration changes without taking
VirtualBox snapshots first. When something broke I had no clean restore point
and had to redo several steps from memory, including Splunk configuration and
network interface settings.

### How I fixed it
Redid the steps — took about an hour total to get back to where I was.

### What I learned
**Take a snapshot before every significant change.** This is not optional.
In VirtualBox: select VM → Machine → Take Snapshot. Name it clearly:
`"clean-kali-splunk-configured"` or `"metasploitable-networking-working"`.

One click saves potentially hours of rework. I now take snapshots at every
stable milestone. This habit directly maps to real SOC and IR work where
preserving system state before making changes is standard procedure.

---

## How I Generally Solved Problems

My process when stuck, in order:

1. **Read the error message carefully** — most of the time the answer is in the
   error itself and I was not reading it closely enough
2. **Google the exact error string** — usually someone has hit it before
3. **Ask an AI** — useful for explaining *why* something works, not just what to run
4. **Try the fix, document what happened** — even if it does not work, knowing
   what did not work is progress

The combination of Google and AI was the most effective. Google finds people who
hit the exact same error. AI helps understand the underlying reason so the same
problem does not happen again.

---

## What I Would Do Differently If Starting From Scratch

**1. Plan the network on paper before touching VirtualBox.**
Drawing the topology first — which VM gets which adapter, what IPs they should
have, what can talk to what — would have prevented most of the networking confusion.
Fifteen minutes of planning saved what ended up being over an hour of debugging.

**2. Take VirtualBox snapshots from the very beginning.**
First snapshot: immediately after a clean OS install, before touching anything.
Second snapshot: after networking is confirmed working.
Third: after Splunk is configured and receiving logs.
Snapshots at stable milestones mean any future breakage is recoverable in seconds.

**3. Set static IPs instead of relying on DHCP.**
DHCP-assigned IPs can change between reboots, which breaks Splunk's syslog
forwarding configuration silently. Static IPs on the Host-Only network would
have been cleaner from day one.

**4. Verify each step before moving to the next.**
I sometimes moved forward assuming a previous step worked, only to discover
later it had not. The habit of verifying — ping test, check the IP, confirm
logs are arriving — before every next step would have saved significant time.

---

## Time to Build

The full lab — VMs configured, networking working, Splunk installed and receiving
logs from Metasploitable — took a few hours end to end across one session.
The majority of that time was spent on the networking issues above, not the
actual tool installation.

This is consistent with real-world experience: infrastructure and connectivity
problems consume more time than the tools themselves.

---

## Planned Future Additions

- [ ] Add Windows 10 VM as a monitored endpoint — Windows Event Logs are the
      primary log source in enterprise SOC environments
- [ ] Set up Wazuh alongside Splunk for XDR/EDR comparison
- [ ] Add pfSense as a network firewall between attacker and target segments
- [ ] Configure static IPs on the Host-Only network to eliminate DHCP issues
