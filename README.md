# Openwrt-or-lede with mwan3, instead of balancing, you can failover as passive-active links
i.e.  mutlihoming soho router w/o BGP

Objective - HA uplinks with failed over, eta 2 minutes+
e.g. Uplink A , 58.65.16.172 (PE)- 58.65.16.173 (CE)
     Uplink B,  58.65.16.174 (PE)- 58.65.16.175 (CE)
     

[Setup your OpenWRT/LEDE router]

Configure or isolate existing port to become WAN2.
Install mwan3 to balance the WAN (both) interfaces.
https://wiki.openwrt.org/doc/howto/mwan3

[Configure "active and passive" interfaces]
1st goto cron https://wiki.openwrt.org/doc/howto/cron, setup it up.
Then you do these bunches of one liners

# Have configured the WAN interfaces as WAN = eth0.2 and WAN2 = eth0.3. 
# Odd timing, detect interface up, add default route

 add these to crontab -e
- 1-59/2 * * * * (ping 58.65.16.172 -c 8 | grep " 0%" && if [ $? -eq 0 ]; then route add default gw 58.65.16.172 ;fi) >/dev/null 2>&1
- 1-59/2 * * * * (ping 58.65.16.174 -c 8 | grep " 0%" && if [ $? -eq 0 ]; then route add default gw 58.65.16.174 ;fi) >/dev/null 2>&1

# Even timing, detect interface down, cut and bring down the interface.
 and these
- */2 * * * * (ping 58.65.16.172 -c 3 | grep " 100%" && if [ $? -eq 0 ]; then ifconfig eth0.2 down; sleep 4; ifconfig eth0.2 up; fi) >/dev/null 2>&1
- */2 * * * * (ping 58.65.16.174 -c 3 | grep " 100%" && if [ $? -eq 0 ]; then ifconfig eth0.3 down; sleep 4; ifconfig eth0.3 up; fi) >/dev/null 2>&1



