aio-nag-st2
==================

Creates an AIO installation of OpenStack, a Nagios monitoring VM and a StackStorm Server using Vagrant and VirtualBox.

Clone the repo and run ***vagrant up*** to create the base VMs.  Linked Clones are used within VirtualBox so the disk space requirements are kept as low as possible.

Warning - the current vagrant file has not been optimised for the Windows version of Vagrant, and has only been tested on OSX - it may need some updates to work on windows hosts.

Once the VMs are online you can build the whole environment up simply running:

**ansible-playbook site.yml**

Alternaitvely you can run the playbooks individually, but run them in the order they are listed in the site.yml file:

- update.yml - updates the base OS imgage
- copy-keys.yml - adds the default public key from your machine to allow easy access to each VM
- restart.yml - restarts the VMs ensuing all updates are activated
- ntp-server.yml - installs ntp server onto the AIO VM
- ntp-client.yml - installs ntp client onto the Nagios and StackStorm VMs
- stackstorm.yml - runs all the roles associated to the StackStorm VM
- aio.yml - runs all the roles associated to the AIO VM and Conatiners - this will take 90 - 120 mins to run!
- nagios.yml - runs all the roles associated to the Nagios VM
- nagios-mon.yml - adds nagios-monitoring to the AIO VM