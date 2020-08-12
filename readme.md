### Wireguard Mesh Network Auto-Generator

This is a utility to auto-generate [wireguard](https://www.wireguard.com/) keys and configuration files to create a
mesh style network of nodes that are connected point-to-point with wireguard.
Each node in the mesh network has a peer connection to all the other nodes.
There is no central server to route network traffic.
I wrote this after manually adding my sixth node to a p2p wireguard network and
thinking that there must be a better way to handle the configuration and all the editing of the files.
The **wg-mesh** python3 script is the result.

System requirements:
- python3 installed and python3 modules installed:
    - os
    - sys
    - configparse
    - argparse
    - ipaddress
    - subprocess
    - tempfile
- [wireguard](https://www.wireguard.com/) installed

It comes with several utility scripts (written for bash) as well:
Script | Arguments | Purpose
------ | --------- | -------
[save\_file](save_file) | filename | Saves a backup version of a file with a yymmdd-n filename extension.  It **moves** the given file to a new name with the extension.  **It does not do a copy.**
[ipv4endpoints](ipv4endpoints) | meshname | Lists out the settings for inet4address of all the nodes.
[ipv6endpoints](ipv6endpoints) | meshname | Lists out the settings for inet6address of all the nodes.
[do-install-files](do-install-files) | meshname | Installs the wireguard configuration files from `etc-wireguard/<node>/` into `/etc/wireguard` on all the nodes.
[do-systemctl](do-systemctl) | meshname command | Will run a systemctl command on all the nodes.  The command must be one of enable, disable, start, or stop.
[do-addconf](do-addconf) | meshname | Will run a `wg addconf` command on all the nodes.  If the `wg addconf` fails then a `systemctl start` is run instead.

#### More requirements
To use wg-mesh you will need to have [wireguard](https://www.wireguard.com/) installed on the system where you run wg-mesh.
It uses the `wg` command to generate private and public keys for the nodes in the network.
It does not need to be part of the mesh network and listed as one of the nodes.
This system will also need to be able to ssh to the root user on each of the nodes in the network.
I do this by setting up an ssh authorized key for the user I'm running wg-mesh as.
Additionally, [wireguard](https://www.wireguard.com/) needs to be installed on all the nodes you will be using in the network.

Having met these basic requirements ... the next steps are:
##### Setup conf/wg-mymesh.ini
Copy the `conf/wg-sample.ini` file to a new file in the conf
directory.<br>
For example, if you choose `mymesh` as a meshname for your network, you would:
```
$ cp conf/wg-sample.ini conf/wg-mymesh.ini
```
Then edit it to suit your needs.
[Documentation for the mesh.ini format is here](doc/wg-meshname.md).
Set the `interface` to what you want to call your wireguard interface.
Add in the node names you want to use for the `nodes` value - one for each system in the network.
The `listenport` should be set to the udp port that you want wireguard to use.
Feel free to adjust network4 and startaddress4 to use what ipv4 private network and addresses you want for this mesh.
The same goes for network6 and startaddress6.
Or - leave these as they are.
Keep in mind if using more than one mesh network that different values will need to be used for a second or
more mesh network.

##### Create a node.ini file for each node
You can use `conf/newnode.ini` as a template for these.
[Documentation for the node.ini format is here](doc/node.md).
Copy this file to `conf/<node>.ini` for each node in your mesh and then edit each of them to suit your configuration.
The section name will need to be changed to match the node name - in the newnode.ini file it is `newnode`.
You will need to change the inet4address in each one to have the actual IPv4 address of the node, or change it
to inet6address if you are using IPv6 in your network.
I prefer to add both when they are available on that node.

##### Run wg-mesh
Now run the `wg-mesh` command with the name of your mesh network as an option -
possibly `./wg-mesh -m mymesh`<br>
Fix any problems encountered.
If it is working normally you should see something like
```
node mynode1 was modified - it needs to be rewritten
node mynode2 was modified - it needs to be rewritten
```
You will find new mynode1.ini and mynode2.ini files in the `etc-wireguard` directory.
There are also mynode1 and mynode2 subdirectories in the `etc-wireguard` directory.
These each contain the wireguard configuration file that will be installed for that node.<br>
**These new files contain wireguard public and private keys - keep them secure.**<br>
The new node config files should replace what we initially created in the `conf` directory.<br>
To do this run the command
```
mv etc-wireguard/*.ini conf/
```
These new configuration files will be used over again (but not modified) if you add a new node to
this same mesh network.
You will also want to preserve the wireguard configuration files in the `conf/<node>/` directories.
I normally just leave them in this directory, but making sure that they are **secure**.

##### Install the configuration files
Now run the do-install-files script `./do-install-files mymesh`<br>
This will do several things for each of the nodes in the mesh network:
1. copy over the save\_file script to /etc/wireguard
1. run the save\_file script on the remote system to save any existing configuration file we will overwrite
1. remove the save\_file script from the remote system
1. copy over the wireguard configuration file that `wg-mesh` created for this node

It will first look for an IPv4 address (inet4address) for the node and use that to work with.
If one is not present in the node configuration file it will try the IPv6 address (inet6address).

##### Tell wireguard about the new configuration
Then run the do-addconf script `./do-addconf mymesh`<br>
This too works a bit like `do-install-files` trying IPv4 first, then IPv6.
It first tries a `wg addconf` command to notify wireguard about the new configuration file.
If that fails, it will try to run a `systemctl start wg-quick@<interface>` command in case wireguard
hasn't been started yet.
The script will tell you up front that you might want to ignore the messages you see from
a failed wg addconf command.  If you know this system has been running wireguard fine and communicating
with other nodes - you may want to investigate the problem.

##### Test
I did not write a script for this but may do so in the near future.
What I normally do to test the configuration is login to the nodes and try to ping each of the
other nodes by their wg4address or wg6address and see that I get a good response to the pings.

##### Enable
To make sure wireguard is enabled for startup at reboot on each of the systems I follow all
this up with the command `./do-systemctl enable`.
This will do as it says - run a `systemctl enable wg-quick@<interface>` command on each of the nodes.

##### To add a new node to an existing mesh network
- Add the node name to the `conf/wg-mymesh.ini` nodes setting
- Create a node.ini file for the new node (see above)
- Run the `./wg-mesh -m mymesh` command.
- Save the new node.ini file: `mv etc-wireguard/*.ini conf/`
- Install the new node wireguard configuration file and all the modidifed files for the other systems: `./do-install-files mymesh`
- Inform wireguard of the new configuarion: `./do-addconf mymesh`
- Test
