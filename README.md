## ZTP upgrading Juniper EX2300 and EX3400 

The purpose of this repo is to enable ZTP-installing Junos on EX2300/EX3400, even when regular ZTP (or manual installation) fails due to lack of space on flash. The process requires no input from the operator. (Almost, I have not automated the final shutdown, as I like to perform a manual sanity check.)

The way it is done, is to use the out-of-box ZTP functionality to load a custom *configuration* file *only*. (I.e. not a Junos upgrade.) 

This config will in turn enable a bit of slax/op-scripting. First, the script does the necessary cleanup prior the Junos upgrade. Then, it kicks off the actual upgrade. After the install completes, the script runs again to check if the correct Junos version is installed. If this is the case, the script disables itself, changes the dhcp vendor-id, and re-enables ZTP. This triggers loading of our preferred deployment config.

All the heavy lifting performed by [kquilliam](https://github.com/kquilliam/juniper-ztp), who (as far as I know) deserves all the glory. This fork is primarily for my own purposes and to add a bit of customization/documentation/explanation.


Slightly more detailed explanation below:

* operator connects EX2300 to power, mgmt (me0) and console
* operator steps away
* first boot:
  * ZTP is enabled out of the box
  * DHCP-server catches default DHCP vendor id, and provides a simplified config file *only* which 
    * disables ZTP
    * loads and runs a slax script
      * slax script cleans up space
      * slax script installs our preferred OS version, which in turn triggers a reboot
* second boot: device boots into new OS
* slax script runs again and
  * finds everything ok (correct OS version is running)
  * makes a snapshot
  * re-enables ZTP
  * sets a custom DHCP vendor id on interface me0
  * disables execution of slax script
* DHCP server catches our custom DHCP vendor-id from client and provides our preferred config *only* for *deployment*
* device applies config, which again disables ZTP (and possibly me0 if so required)
* operator returns and verifies that device runs required OS version and deployment config
* operator shuts down device and ships it for deployment



## Prerequisites

* ISC DHCP server, with custom dhcpd.conf
* httpd, hosting:
  * slax enabled config
  * your preferred final config
  * slax script
  * preferred OS versions to be installed

Sample files provided in this repo.


## Usage

* create an isolated VLAN for everything ZTP, put N switchports in said VLAN (configured as access-ports)
* customize dhcpd.conf as you see fit
* fire up your dhcpd with said config file
* customize your ztp.slax file as you see fit
* make sure ztp.slax, config files referenced in dhcpd.conf and OS images referenced in ztp.slax all are available via http
* connect switch management port to your ZTP vlan
* connect power to switch


## Troubleshooting

* is the device fresh out of box? 'request system zeroize' if not
* be patient.
  * booting Junos on EX is slow
  * the os installation takes time
  * the final run of the slax script also does a snapshot

* run dhcpd in the foreground with flag -d. Like this:
```dhcp-server # /usr/sbin/dhcpd -4 -cf /etc/dhcp/dhcpd.conf -d```
* the slax script provides some output in /var/log/op*
* the slax script can be run manually:
```switch # run op url http:/server/path/to/scriptname```
* connect to the switch console
* watch the httpd log file
 ```httpd # tail -f /var/log/httpd/access_log```
* if the device fails to boot, it can be reinstalled via usb or via bootloader/tftp. You need particular OS-images for this.


## Gotchas/missing features/disclaimer
* this script does not upgrade the PoE firmware automatically. You want to do that prior to deployment...
* if anything breaks by following these instructions, you get to keep all the pieces. I assume no responsibility of any kind whatsoever.
