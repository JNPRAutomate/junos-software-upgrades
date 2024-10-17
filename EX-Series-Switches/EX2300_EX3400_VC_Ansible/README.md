## Ansible Upgrades for EX2300 and EX3400 Virtual Chassis

### Recommended Information to Provide:

Hardware model: EX2300 and EX3400, standalone or in a Virtual Chassis Configurartion

### Ansible Software

These playbook used the Juniper Junos role: 
https://galaxy.ansible.com/ui/standalone/roles/juniper/junos/

This Ansible role is superseded by the Juniper Device Collection: 
https://galaxy.ansible.com/ui/repo/published/juniper/device/

### Usage ###

These are a couple of playbooks that were taken from a series used in a ZTP process to bring up and configure
EX2300s and EX3400s used as access switches in a campus network.

There were serveral challenges encountered in getting the upgrade process to go smoothly and consistently on
these devices, especially when they were in a Virutal Chassis configuration.  

The first issue, was the ever present lack of storage space on these two platforms. The solution for this 
was running a SLAX script (git submodule) called free_up_space.slax which clears out as much space as possible
from every memmber of a virtual chassis, hopefully leaving enough room for the upgrade to complete successfully.

The second issue was time, and managing timeouts.  Upgrading a EX3400, espeically a VC can take a VERY long time, 
and the bultin liveliness timers that ansible had kept timing out at various stages.  This issue came with two of
it's own challenges, optimizing the way the upgrades were performed and the timer tweaks at the various stages
of the upgrade process.  During my testing, at least on Junos versions from 18 to 20, scp'ing the Junos images to 
the EX3400s was incredibly slow, taking on the order of 30 to 40 minutes for a 300 Mbyte file.  Avoiding the encryption
by using HTTP or FTP for the transfer sped things up 5 fold.  Also, if upgrading a VC using a URL, each VC member
fetched it's own copy independently, which really slowed things down waitng for more transfers.  The key to avoiding
this was to get a local copy of Junos on the master image, which would then be pushed to the VC members from the master
RE (VC member) as they upgraded.  I wound up temporarily enabling FTP on the devices being upgraded, and then using
curl on the ansible host to copy the Junos image to the devices, and then turning FTP back off.  

The timers issue was a tough one, because if they were too small the playbook failed...and if they were too large then
the playbook would have a lot of idle time.  I wound up taking a middle ground on timeout values in ansible (ansible.cfg) and 
with playbook tasks, and looping back with a "checking" task in the playbook to check on livlieness so the playbook 
could move onto the next task as quickly as possible.

The upgrade_001_free_up_space.yml also needs a functioning HTTP server, used to remotly run free_up_space.slax
as an op script.  

Junos images for various platforms, as well as some other timeout functions are managed from the junos.yml file in 
the group_vars folder.
