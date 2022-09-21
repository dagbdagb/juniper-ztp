The purpose of this repo is to enable ZTP-installing Junos on EX2300/EX3400, even when installation fails due to lack of space.
The way it is done, is to use the out-of-box ZTP functionality to load a custom *configuration* file *only*. (I.e. not a Junos upgrade.) 

This config will in turn enable a bit of slax/op-scripting, which does the cleanup necessary for installing Junos. After the install, the script runs again to check if the correct Junos version is installed. If this is the case, the script disables itself.

All the heavy lifting performed by [kquilliam](https://github.com/kquilliam/juniper-ztp), who deserves all the glory. This fork is primarily for my own purposes and to add a bit of customization/documentation/explanation.


Slightly more detailed explanation:

* operator connects EX2300 to power, mgmt (me0) and console
* operator steps away
* first boot:
  * ZTP is enabled out of the box
  * DHCP-server catches default DHCP vendor id, and provides a simplified config file *only* which 
    * disables ZTP
    * loads and runs a slax script
      * slax script cleans up space
      * slax script installs our preferred OS version
* device boots into new OS
* slax script runs again and
  * finds everything ok
  * re-enables ZTP
  * sets a custom DHCP vendor id on interface me0
  * disables execution of slax script
* DHCP catches custom DHCP vendor id and provides our default config *only* for deployment
* device applies config
* operator verifies that device runs required os version and config
* operator shuts down device and ships it for deployment



Prerequisites:

* ISC DHCP server, with custom dhcpd.conf
* httpd, hosting:
  * slax enabled config
  * slax script
  * preferred deployment config
  * preferred OS versions to be installed


