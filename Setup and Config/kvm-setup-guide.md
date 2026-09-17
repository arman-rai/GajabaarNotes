# KVM Setup Guide — LAN Bridging over WiFi (VMware-style)
> Created: 2026-09-16 | Host: Fedora | User: namura

---

## Overview

This guide documents how to give KVM virtual machines real `192.168.0.x` LAN IPs
over WiFi — replicating VMware Workstation's `vmnet0` bridged mode behaviour,
which internally uses **Proxy ARP** rather than true bridging.

---

## Why Standard Bridging Fails on WiFi

Linux WiFi drivers (802.11) do **not** support bridge enslaving. The protocol
requires all frames to carry the AP's MAC address — a bridge needs to forward
frames with arbitrary VM MAC addresses, which the driver rejects at the kernel level.

VMware works around this using its proprietary `vmnet.ko` kernel module, which
intercepts packets and uses **Proxy ARP** at layer 3 instead of bridging at layer 2.

---

## Architecture — Proxy ARP Method

```
Router (192.168.0.1)
       │
  WiFi (wlp0s20f3) — proxy_arp=1
       │   ↑
       │   └─ Host answers ARP for VM IPs using its own WiFi MAC
       │      "Who has 192.168.0.200?" → host replies → router sends to host
       │
  [Linux kernel ip_forward=1]
       │
  virbr1 (192.168.0.193/26) — proxy_arp=1
       │
  ┌────┴────┐
 VM1       VM2
192.168.0.194  192.168.0.195
(DHCP from host)
```

**Key principle:** VMs get real LAN IPs. The host proxies ARP on their behalf
so the router and other LAN devices can reach them directly.

---

## System Requirements

| Component | Value |
|---|---|
| Host OS | Fedora (any modern) |
| Hypervisor | KVM/QEMU via libvirt |
| WiFi interface | `wlp0s20f3` |
| Host IP | `192.168.0.108` |
| Router | `192.168.0.1` |
| VM subnet | `192.168.0.192/26` (`.194–.250` for VMs) |

---

## Step 1 — Install Required Packages

```bash
sudo dnf install virt-manager virt-viewer qemu-kvm libvirt spice-vdagent
sudo systemctl enable --now libvirtd
```

---

## Step 2 — Add User to libvirt Group

```bash
sudo usermod -aG libvirt,kvm namura
# Log out and back in after this
```

> **Important:** Always use `virsh -c qemu:///system` or open virt-manager
> normally. User-session virsh (`qemu:///session`) lacks privileges to create
> bridge interfaces.

---

## Step 3 — Create the Proxy ARP Libvirt Network

Save the following as `~/proxyarp-lan.xml`:

```xml
<network>
  <name>proxyarp-lan</name>
  <forward mode="route" dev="wlp0s20f3"/>
  <bridge name="virbr1" stp="off" delay="0"/>
  <ip address="192.168.0.193" netmask="255.255.255.192">
    <dhcp>
      <range start="192.168.0.194" end="192.168.0.250"/>
    </dhcp>
  </ip>
</network>
```

Then define, autostart, and start it:

```bash
virsh -c qemu:///system net-define ~/proxyarp-lan.xml
virsh -c qemu:///system net-autostart proxyarp-lan
virsh -c qemu:///system net-start proxyarp-lan
virsh -c qemu:///system net-list --all
```

Expected output:
```
 Name           State    Autostart   Persistent
-------------------------------------------------
 default        active   yes         yes
 proxyarp-lan   active   yes         yes
```

---

## Step 4 — Enable Proxy ARP (Persistent)

Save as `/etc/sysctl.d/99-kvm-proxyarp.conf`:

```ini
net.ipv4.conf.wlp0s20f3.proxy_arp = 1
net.ipv4.conf.virbr1.proxy_arp = 1
net.ipv4.ip_forward = 1
```

Apply immediately:

```bash
sudo cp ~/99-kvm-proxyarp.conf /etc/sysctl.d/99-kvm-proxyarp.conf
sudo sysctl --system
```

Verify:

```bash
sysctl net.ipv4.conf.wlp0s20f3.proxy_arp   # should be 1
sysctl net.ipv4.conf.virbr1.proxy_arp       # should be 1
```

---

## Step 5 — Ethernet Bridge (Optional, for when cable is plugged in)

When ethernet (`enp7s0`) is available, VMs can use a true bridge instead,
getting DHCP directly from the router:

```bash
nmcli con add ifname br0 type bridge autoconnect yes con-name br0 \
  ipv4.method auto bridge.stp no
nmcli con add type bridge-slave autoconnect yes \
  con-name br-slave-enp7s0 ifname enp7s0 master br0
nmcli connection up br0
```

Libvirt network for the ethernet bridge:

```xml
<network>
  <name>host-bridge</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>
```

```bash
virsh -c qemu:///system net-define ~/host-bridge.xml
virsh -c qemu:///system net-autostart host-bridge
virsh -c qemu:///system net-start host-bridge
```

---

## Step 6 — Migrate VMware Kali VM to KVM

```bash
mkdir -p ~/kvm

# Convert split VMDKs to qcow2 (point at the descriptor .vmdk, not -s001 etc.)
qemu-img convert -f vmdk -O qcow2 \
  ~/vmware/KaliLinuxV2/KaliLinuxV2.vmdk \
  ~/kvm/KaliLinuxV2.qcow2
```

> Shut down the VMware VM completely before converting.

---

## Step 7 — Create VM in virt-manager

1. **New VM** → **Import existing disk image**
2. Browse to `~/kvm/KaliLinuxV2.qcow2`
3. OS: Debian 11/12 (closest to Kali)
4. RAM: `2048 MB`, CPUs: `2`
5. ✅ Tick **"Customize configuration before install"**
6. **NIC** → Network source: `proxyarp-lan`
7. **Display** → `Spice`
8. **Video** → `QXL`
9. Click **Begin Installation**

---

## Step 8 — SPICE Clipboard (Inside the Guest VM)

No VMware Tools needed. Just install the SPICE agent:

```bash
# Inside Kali/Debian VM
sudo apt install spice-vdagent
sudo systemctl enable --now spice-vdagentd
```

Clipboard now works **bidirectionally** and seamlessly — including with
clipboard managers like CopyQ on the host.

---

## Network Reference

| Network | Type | VMs get | Internet | Host↔VM | Requires |
|---|---|---|---|---|---|
| `proxyarp-lan` | Route + Proxy ARP | `192.168.0.194–250` | ✅ | ✅ | WiFi only |
| `host-bridge` | True bridge | `192.168.0.x` (router DHCP) | ✅ | ✅ | Ethernet cable |
| `default` (virbr0) | NAT | `192.168.122.x` | ✅ | ✅ | Nothing |

---

## Troubleshooting

### VM doesn't get a 192.168.0.x IP
- Check proxy ARP is active: `sysctl net.ipv4.conf.wlp0s20f3.proxy_arp`
- Check virbr1 is up: `ip addr show virbr1`
- Make sure VM network is set to `proxyarp-lan` in virt-manager
- Verify the VM is using DHCP on its NIC

### virsh: "Operation not permitted"
- Use `virsh -c qemu:///system` not plain `virsh`
- Ensure you're in the `libvirt` group and have re-logged in

### Clipboard not working in KVM
- Install `spice-vdagent` inside the guest
- Ensure Display is set to **Spice** and Video to **QXL** in virt-manager

### "Network not found" error
- Network was defined in user session, not system: re-define with `virsh -c qemu:///system`

---

## File Locations

| File | Purpose |
|---|---|
| `~/proxyarp-lan.xml` | Libvirt network definition |
| `~/99-kvm-proxyarp.conf` | Sysctl proxy ARP config (copy to `/etc/sysctl.d/`) |
| `~/kvm/KaliLinuxV2.qcow2` | Converted Kali VM disk |
| `/etc/sysctl.d/99-kvm-proxyarp.conf` | Persistent proxy ARP (survives reboot) |
