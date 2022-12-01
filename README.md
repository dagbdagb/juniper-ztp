## ZTP upgrading Juniper EX2300 and EX3400 

The purpose of this repo is to enable ZTP-installing Junos on EX2300/EX3400, even when regular ZTP (or manual installation) fails due to lack of space on flash. The process requires no input from the operator. (Almost, I have not automated the final shutdown, as I like to perform a manual sanity check.)

The way it is done, is to use the out-of-box ZTP functionality to load a custom *configuration* file *only*. (I.e. not a Junos upgrade.) 

This config will in turn enable a bit of slax/op-scripting. First, the script does the necessary cleanup prior to the Junos upgrade. Then, it kicks off the actual upgrade. After the install completes, the script runs again to check if the correct Junos version is installed. If this is the case, the script disables itself, changes the dhcp vendor-id, and re-enables ZTP. This triggers loading of our preferred deployment config.

All the heavy lifting performed by [kquilliam](https://github.com/kquilliam/juniper-ztp), who (as far as I know) deserves all the glory. This fork is primarily for my own purposes and to add a bit of customization/documentation/explanation.


## Procedure explained

* operator connects EX2300 to power, mgmt (me0) and console
* *if* device isn't fresh out of box, wait for device to complete booting. then: ```request system zeroize```
* step aside, get coffee
* first boot:
  * ZTP is enabled out of the box
  * DHCP-server catches DHCP vendor id from Junos default config, and provides a simplified config file *only* which 
    * disables ZTP
    * loads and runs a slax script, which
      * cleans up space on flash
      * installs our preferred OS version, which in turn triggers a reboot
* second boot: device boots into new OS
* slax script runs again and
  * finds everything ok (correct Junos version is running)
  * makes a snapshot
  * re-enables ZTP
  * sets a custom DHCP vendor id on interface me0
  * disables execution of slax script
* DHCP server catches our custom DHCP vendor-id from the Junos DHCP-client and provides (again, via DHCP option 43) the name and location of our custom config *only* for *deployment*
* device applies our deployment config, which again disables ZTP (and possibly me0 if so configured in the custom deployment config)
* operator returns and verifies that device runs required OS version and deployment config
* operator shuts down device and ships it for deployment



## Prerequisites

* ISC DHCP server, with custom dhcpd.conf (sample provided)
* httpd, hosting:
  * slax enabled config (sample provided)
  * your preferred final config (sample provided)
  * slax script (provided, modify as required)
  * preferred OS versions to be installed (via your Juniper support agreement)



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
* this script does not upgrade the PoE firmware automatically, if required:
```
root@unconfigured-ex2300-48> show poe controller 
Controller  Maximum   Power         Guard    Management   Status        Lldp
index       power     consumption   band                                Priority 
   0**      750W      0.00W           0W     Class        AT_MODE       Disabled
  **New PoE software upgrade available. 
 Use 'request system firmware upgrade poe fpc-slot <slot>' 
 This procedure will take around 10 minutes (recommended to be performed during maintenance)

{master:0}
```
You really want to perform this upgrade prior to deployment... and I should incorporate it into the slax script. Some time. 

Historically, there was at least one nasty bug related to this procedure, rendering the PoE controller dead to the point of having to RMA the entire switch. 

Anyways, for now:
```
root@unconfigured-ex2300-48> request system firmware upgrade poe fpc-slot 0
root@unconfigured-ex2300-48> request system reboot in 30 
Reboot the system in 30? [yes,no] (no) yes 
```
Then *walk away* from the switch. Do nothing more until it has finished rebooting. If you have a VC (stack of multiple switches as one logical unit), do *one* PoE firmware at a time.
* if anything breaks by following these instructions, you get to keep all the pieces. I assume no responsibility of any kind whatsoever.
