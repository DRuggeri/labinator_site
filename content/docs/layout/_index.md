---
Title: Layout and Systems
Weight: 3
---
<style>
h3 {
    clear: both;
    padding-top: 1em;
}
</style>

Labinator is made from a bunch of cheap parts bolted to a board, but what is interesting is how all of the parts come together to do fun stuff. On this page you will find some details about where each of the systems physical sit on the Labinator board, how they are physically wired, and how they are configured.

{{<image path="images/layout.png" width=720 alt="Labinator Layout" >}}

The images trace cables with the color key of <span style="color:red">power</span>, <span style="color:green">network</span>, <span style="color:blue">power toggle</span>.

### Wally
{{<image path="images/layout-export-wally.png" height=250 class="layoutpic">}}
* IP: 192.168.122.1
* Power: 5v port 1
* Net: Port 8
* OS: OpenWRT
* Location: Under top touchscreen
{{% details title="Configuration" %}}
* Passwords
  * root - `root:wally`
```bash
screen /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A5069RR4-if00-port0 1500000
root
password

# Reset root password
echo 'wally
wally' | passwd

# Add post-init scripts here
echo '
' > /etc/rc.local

# Packages
if [ `opkg list-installed | grep -c prometheus` -lt 1 ];then
  opkg install prometheus-node-exporter-lua prometheus-node-exporter-lua-nat_traffic prometheus-node-exporter-lua-netstat prometheus-node-exporter-lua-openwrt
fi

if [ `opkg list-installed | grep -c luci-mod-rpc` -lt 1 ];then
  opkg install luci-mod-rpc
fi

# Login
uci set dropbear.@dropbear[0].PasswordAuth='on'
uci set dropbear.@dropbear[0].RootPasswordAuth='on'

# Set LAN IP and disable dhcp on LAN
uci set network.lan.ipaddr='192.168.122.1'
uci set dhcp.lan.ignore='1'

# Ensure WAN is DHCP and LAN DHCP is off
uci set network.wan.proto='dhcp'

# Set system params
uci set system.@system[0].hostname='wally'
uci set system.@system[0].zonename='UTC'
uci delete system.@system[0].timezone
uci set system.ntp=timeserver
uci set system.ntp.use_dhcp='0'
uci set system.ntp.server='192.168.122.3'

# Syslog to boss
uci set system.@system[0].log_proto='tcp'
uci set system.@system[0].log_port='601'
uci set system.@system[0].log_ip='192.168.122.3'
uci set system.@system[0].conloglevel='8'
uci set system.@system[0].cronloglevel='5'

# Default policies - LAN
uci set firewall.@zone[0].input='DROP'
uci set firewall.@zone[0].output='ACCEPT'
uci set firewall.@zone[0].forward='ACCEPT'
uci set firewall.@zone[0].log='1'

# Default policies - WAN
uci set firewall.@zone[1].input='DROP'
uci set firewall.@zone[1].output='ACCEPT'
uci set firewall.@zone[1].forward='DROP'
uci set firewall.@zone[1].log='1'


uci set network.wwan=interface
uci set network.wwan.proto='dhcp'

uci set firewall.wwan=zone
uci set firewall.wwan.name='wwan'
uci set firewall.wwan.masq='1'
uci set firewall.wwan.log='1'
uci set firewall.wwan.input='DROP'
uci set firewall.wwan.output='ACCEPT'
uci set firewall.wwan.forward='DROP'
uci set firewall.wwan.network='wwan'

uci set wireless.radio0=wifi-device
uci set wireless.radio0.type='mac80211'
uci set wireless.radio0.path='platform/usbhost/fd000000.usb/xhci-hcd.0.auto/usb1/1-1/1-1:1.0'
uci set wireless.radio0.channel='auto'
uci set wireless.radio0.band='5g'
uci set wireless.radio0.country='US'
uci set wireless.radio0.mode='sta'
uci set wireless.radio0.cell_density='1'
uci del wireless.radio0.disabled
uci del wireless.radio0.htmode

uci set wireless.wifi0=wifi-iface
uci set wireless.wifi0.device='radio0'
uci set wireless.wifi0.mode='sta'
uci set wireless.wifi0.network='wwan'
uci set wireless.wifi0.ifname='wifi0'
uci set wireless.wifi0.encryption='psk2'
uci set wireless.wifi0.ssid='Danimal-Guest'
uci set wireless.wifi0.key='REDACTED'
### Restart radio0

uci set network.wwan.device='wifi0'
### Restart wwan network interface

# Allow Web UI on all interfaces
uci add firewall rule
uci set firewall.@rule[-1].name='Web-UI'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='*'
uci set firewall.@rule[-1].dest_port='443'
uci set firewall.@rule[-1].target='ACCEPT'

# Allow SSH on LAN interface
uci add firewall rule
uci set firewall.@rule[-1].name='LAN-SSH'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='lan'
uci set firewall.@rule[-1].dest_port='22'
uci set firewall.@rule[-1].target='ACCEPT'

# Allow port 22 on outside for port forward to boss
uci add firewall rule
uci set firewall.@rule[-1].name='Boss-SSH'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].dest_port='22'
uci set firewall.@rule[-1].target='ACCEPT'
uci add firewall redirect
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].target='DNAT'
uci set firewall.@redirect[-1].name='Boss SSH'
uci set firewall.@redirect[-1].proto='tcp'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='22'
uci set firewall.@redirect[-1].dest_ip='192.168.122.3'

# Grafana port 3000 to boss
uci add firewall rule
uci set firewall.@rule[-1].name='Grafana'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].dest_port='3000'
uci set firewall.@rule[-1].target='ACCEPT'
uci add firewall redirect
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].target='DNAT'
uci set firewall.@redirect[-1].name='Boss Grafana'
uci set firewall.@redirect[-1].proto='tcp'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='3000'
uci set firewall.@redirect[-1].dest_ip='192.168.122.3'

# Loki port 3100 to boss
uci add firewall rule
uci set firewall.@rule[-1].name='Loki'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].dest_port='3000'
uci set firewall.@rule[-1].target='ACCEPT'
uci add firewall redirect
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].target='DNAT'
uci set firewall.@redirect[-1].name='Boss Loki'
uci set firewall.@redirect[-1].proto='tcp'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='3100'
uci set firewall.@redirect[-1].dest_ip='192.168.122.3'

# VNC 5900 to boss
uci add firewall rule
uci set firewall.@rule[-1].name='VNC'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].dest_port='5900'
uci set firewall.@rule[-1].target='ACCEPT'
uci add firewall redirect
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].target='DNAT'
uci set firewall.@redirect[-1].name='Boss VNC'
uci set firewall.@redirect[-1].proto='tcp'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='5900'
uci set firewall.@redirect[-1].dest_ip='192.168.122.3'

# Prometheus 9090 to boss
uci add firewall rule
uci set firewall.@rule[-1].name='Prometheus'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].dest_port='9090'
uci set firewall.@rule[-1].target='ACCEPT'
uci add firewall redirect
uci set firewall.@redirect[-1].dest='lan'
uci set firewall.@redirect[-1].target='DNAT'
uci set firewall.@redirect[-1].name='Boss Prometheus'
uci set firewall.@redirect[-1].proto='tcp'
uci set firewall.@redirect[-1].src='wan'
uci set firewall.@redirect[-1].src_dport='9090'
uci set firewall.@redirect[-1].dest_ip='192.168.122.3'

# VNC to the hosts
for i in $(seq 1 6);do
  IP="192.168.122.1${i}"
  for j in $(seq 1 9);do
    PORT="59${i}${j}"
    VNCPORT=$((5900 + j))
    uci add firewall rule
    uci set firewall.@rule[-1].name="Lab VNC $PORT"
    uci set firewall.@rule[-1].proto="tcp"
    uci set firewall.@rule[-1].src="wan"
    uci set firewall.@rule[-1].dest_port="$PORT"
    uci set firewall.@rule[-1].target="ACCEPT"
    uci add firewall redirect
    uci set firewall.@redirect[-1].dest="lan"
    uci set firewall.@redirect[-1].target="DNAT"
    uci set firewall.@redirect[-1].name="Lab VNC $PORT"
    uci set firewall.@redirect[-1].proto="tcp"
    uci set firewall.@redirect[-1].src="wan"
    uci set firewall.@redirect[-1].src_dport="$PORT"
    uci set firewall.@redirect[-1].dest_ip="$IP"
    uci set firewall.@redirect[-1].dest_port="$VNCPORT"
  done
done

# Node exporter
uci set prometheus-node-exporter-lua.main.listen_interface='lan'
uci set prometheus-node-exporter-lua.main.listen_port='9100'

uci add firewall rule
uci set firewall.@rule[-1].name='Node Exporter'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].src='lan'
uci set firewall.@rule[-1].dest_port='9100'
uci set firewall.@rule[-1].target='ACCEPT'


uci commit
reboot -f
```
{{% /details %}}

### Boss
{{<image path="images/layout-export-boss.png" height=250 class="layoutpic">}}
* IP: 192.168.122.3
* Power: 12v port 10
* Relay: Port 7
* Net: Port 7
* OS: Debian GNU Linux
* Location: Under bottom touchscreen
{{% details title="Ports" %}}
* USB side, top: USB Hub
  * Hub port 1: Wally FTDI (top port)
  * Hub port 2: Relayinator FTDI
  * Hub port 3: Statusinator USB
  * Hub port 4: Power for VGA -> HDMI adapter (bottom port)
* USB side, bottom: TP-Link UE306 GBe
* USB top, top: Top screen touch interface
* USB top, bottom: Bottom screen touch interface
* Serial:
  * Wally: /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A5069RR4-if00-port0
  * Relayinator: /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A50285BI-if00-port0
  * Statusinator: /dev/serial/by-id/usb-Espressif_USB_JTAG_serial_debug_unit_98:3D:AE:E9:29:08-if00
{{% /details %}}
{{% details title="Configuration" %}}
* Exposed Ports
  * 22/tcp: ssh
  * 3000/tcp: grafana UI
  * 3100/tcp: grafana Loki
  * 5900/tcp: VNC
  * 9090/tcp: prometheus
* Passwords
  * users
    * `root:???`
    * `boss:???`
  * grafana - `admin:admin`
{{% /details %}}


### Switch
{{<image path="images/layout-export-switch.png" height=250 class="layoutpic">}}
* IP: 192.168.2.2
* Power: 12v port 2
* Net: N/A
* OS: N/A
* Location: Bottom right of lab
{{% details title="Configuration" %}}
* Passwords
  * `admin:admin`
{{% /details %}}


### Relayinator
{{<image path="images/layout-export-relayinator.png" height=250 class="layoutpic">}}
* IP: N/A
* Power: 12v hardwired
* Net: N/A
* OS: [custom](/docs/subprojects/relayinator/)
* Location: Center bottom of lab


### Statusinator
{{<image path="images/layout-export-statusinator.png" height=250 class="layoutpic">}}
* IP: N/A
* Power: 5v USB from Boss
* Net: N/A
* OS: [custom](/docs/subprojects/statusinator/)
* Location: Bottom right atop Switch


### Node 1
{{<image path="images/layout-export-node1.png" height=250 class="layoutpic">}}
* IP: 192.168.122.11
* Power: 12v Port 4
* Relay: Port 1
* Net: Port 6


### Node 2
{{<image path="images/layout-export-node2.png" height=250 class="layoutpic">}}
* IP: 192.168.122.12
* Power: 12v Port 5
* Relay: Port 2
* Net: Port 5


### Node 3
{{<image path="images/layout-export-node3.png" height=250 class="layoutpic">}}
* IP: 192.168.122.13
* Power: 12v Port 6
* Relay: Port 3
* Net: Port 4


### Node 4
{{<image path="images/layout-export-node4.png" height=250 class="layoutpic">}}
* IP: 192.168.122.14
* Power: 12v Port 7
* Relay: Port 4
* Net: Port 3


### Node 5
{{<image path="images/layout-export-node5.png" height=250 class="layoutpic">}}
* IP: 192.168.122.15
* Power: 12v Port 8
* Relay: Port 5
* Net: Port 8


### Node 6
{{<image path="images/layout-export-node6.png" height=250 class="layoutpic">}}
* IP: 192.168.122.16
* Power: 12v Port 9
* Relay: Port 6
* Net: Port 7
