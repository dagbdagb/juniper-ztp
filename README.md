The purpose of this repo is to enable ZTP-installing Junos on EX2300/EX3400, even when installation fails due to lack of space.
The way it is done, is to use the out-of-box ZTP functionality to load a custom *configuration* file *only*. (I.e. not a Junos upgrade.) 

This config will in turn enable a bit of slax/op-scripting, which does the cleanup necessary for installing Junos. After the install, the script runs again to check if the correct Junos version is installed. If this is the case, the script disables itself.

All the heavy lifting performed by [kquilliam](https://github.com/kquilliam/juniper-ztp), who deserves all the glory. This fork is primarily for my own purposes and to add more documentation/explanation.


Prerequisites:

- isc-dhcp-server
- vsftpd

The following JUNOS software will need to be downloaded and placed in /var/tmp/ztp/code/:

- EX2300/3400: 18.2R3-S2.9
