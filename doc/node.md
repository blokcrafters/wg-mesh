### A WG-Mesh Node Config File
The node config file is handled using the python3 configparser module documented
[here](https://docs.python.org/3/library/configparser.html).

Be aware that the public and private values in this file are wireguard public and private keys for this node
and need to be kept **secure** and **not** checked into a **public** source code repository.

Here is what a sample wg-mesh node.ini file looks like:
```
[nodename]
# A WG-Mesh Node Config File
# inet4address or inet6address is required; both are allowed
# listenport is optional; the default value from the wg-<meshname>.ini file will be used if not set
# public and private will be auto-generated when not present
# wg4address will be auto-generated when not present and inet4address is present
# wg6address will be auto-generated when not present and inet6address is present
inet4address = 192.0.2.1
inet6address = 2001:db8:200:e70::
listenport = 24825
public = LqH1uDsz+YqTtlWDRS4nmb7hzDUM/Ocf1kCqTgbPSFw=
private = 4Ne3Zs43adLWsrmpG0AZ75aw9UGP6u7r6DWk6ZU35Fw=
wg4address = 10.0.0.1
wg6address = fd86:ea04:1115::1
```
The first line is the [nodename] and the `nodename` should be your name for this particular wireguard node within the network.
It can be nearly any name you would like to use.  It's probably best to keep it reasonable - letters, digits, dashes, underscores,
etc.  It is also used as part of the node config file name so must conform to the file naming requirements (no slashes, colons,
etc in it) for your system.
The config file itself should be named `nodename.ini`.  The nodename part of the filename must match the
section name in the file.

When starting a new wireguard node and creating the node config file for it the minimum and recommended setting is the
`nodename` itself and then at least either the inet4address or the inet6address.  If both are available for this node then
I recommend putting both into the config file.  The only other minimum requirement for the file is a listenport value.
I recommend setting the listenport value network wide for this particular mesh network in the wg-\<meshname\>.ini config file instead.
The listenport value from the wg-\<meshname\>.ini will be used as the default value for any nodes in that mesh that do not
have a listenport value in their node.ini file.
The rest of the values that are needed in the node.ini config file will then be auto-generated for the node by
wg-mesh and the file re-written.

**You will want to keep these files stored away somewhere (version control) and also secure.
Be very careful to keep these secure - they have wireguard public and private keys in them.**
