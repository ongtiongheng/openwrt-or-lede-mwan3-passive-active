# openwrt-or-lede-mwan3-passive-active
A working active-passive mutlihoming soho router w/o BGP

Simple setup your OpenWRT/LEDE router

Configure or isolate existing port to become WAN2.
Install mwan3 to balance the WAN (both) interfaces.
https://wiki.openwrt.org/doc/howto/mwan3

Configure "active and passive" interfaces
Goto cron https://wiki.openwrt.org/doc/howto/cron, setup it up.


- # Have configured the WAN interfaces as WAN = eth0.2 and WAN2 = eth0.3. 
- # Odd timing, detect interface up, add default route
- 1-59/2 * * * * (ping 2.2.2.2 -c 8 | grep " 0%" && if [ $? -eq 0 ]; then ifconfig eth0.2 up; route add default gw 2.2.2.2 ;fi) >/dev/null 2>&1
- 1-59/2 * * * * (ping 3.3.3.3 -c 8 | grep " 0%" && if [ $? -eq 0 ]; then ifconfig eth0.3 up; route add default gw 3.3.3.3 ;fi) >/dev/null 2>&1
- # Even timing, detect interface down, cut and bring down the interface.
- */2 * * * * (ping 2.2.2.2 -c 3 | grep " 100%" && if [ $? -eq 0 ]; then ifconfig eth0.2 down ;fi) >/dev/null 2>&1
- */2 * * * * (ping 3.3.3.3 -c 3 | grep " 100%" && if [ $? -eq 0 ]; then ifconfig eth0.3 down ;fi) >/dev/null 2>&1


