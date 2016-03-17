# Topology Converter
=====================

## **See the WIKI page for more info!!!**
https://wiki.cumulusnetworks.com/display/~eric/Using+Topology+Converter


**Changelog:**

* v1\.0 2015\_10\_19 Initial version constructed
* v2\.0 2016\_01\_07 Added Support for MAC handout, empty ansible playbook (used to generate ansible inventory), [EMPTY] connections for more accurate automation simulation, 
"vagrant" interface remapping for hosts and switches, warnings for interface reuse, added optional support for OOB switch.
* v2\.1 2016\_01\_15 Added Support Boot Ordering -- 1st device --> 2nd device --> servers --> switches
* v2\.2 2016\_01\_19 Added Support for optional switch interface configuration
* v2\.5 2016\_01\_25 Added libvirt support :) and added support for fake devices!
* v2\.6 2016\_01\_26 Moved provider and switch/server mem settings from topology_converter to definitions.py
* v2\.7 2016\_01\_27 Setup cleanup of remap_files added zip file for generated files
* v2\.8 2016\_02\_05 Added support for .def files along with definitions.py so seperate files can be stored in the same directory. Also added support for adding topology files to shareable zipfile.
* v3\.0 2016\_02\_23 Added support for Interface Remapping without reboots on Vx and Hosts (to save time). Moved any remaining topology-specific settings into the definitions files. So topology_converter is truly agnostic and should not need to be modified. Also created an option to disable the vagrant synced folder to further speed boot. Hardened Interface remapping on hosts to work on reboots; and not to pause and wait for networking at reboot. Created remap_eth_swp script that is both hardened and works for both Vx nodes and generic hosts.
* v3\.1 2016\_03\_03 Added Hidden "use_vagrant_interface" option to optionally use Vagrant interface. Added CLI for Debugging mode.
* v3\.2 2016\_03\_08 Significant changes to surrounding packages i.e. rename_eth_swp script. Minor changes to topology converter in the way remap files are generated and hosts run the remap at reboot via rc.local.


# Features:

* Converts "topology.dot" file and associated definition file "topology.def" into a Vagrantfile
* Supports the Virtualbox and Libvirt Vagrant Providers
* Handles interface remapping on Vx instances (and hosts) to match the interfaces used in the provided topology file
* Removes extra Ruby-based logic from the Vagrantfile making a simple human-readable output
* Can handle combined Vagrantfile Generation for Servers and Switches
* "Simple" standalone python script format, does not require Vagrant/Virtualbox/libvirt to be installed to create the Vagrantfile
* 2 files are modified by user to create a suitable Vagrantfile (topology.dot and topology.def)

# Glossary:
**Topology File** -- Usually a file ending in the ".dot" suffix. This file describes the network topology link-by-link. Written in https://en.wikipedia.org/wiki/DOT_(graph_description_language)

**Definitions File** -- This file provides additional information about the topology to be generated. It is actually written in python. It should share the exact same name as the topology file however with the ".def" suffix instead of the ".dot" suffix used by the topology file. (Legacy versions of the definition file used the definitions.py file instead for all topology files)

# To Run: 
Run topology_converter against a topology file: 

```
    $ python ./topology_converter.py ./examples/2switch.dot
```

OR

```
    $ python ./topology_converter.py ./examples/reference_topology_virtualbox.dot
```

OR

```
    $ python ./topology_converter.py ./examples/cw_4switch_2server.dot
```


# What is happening?

1. When topology_converter (TC) is called, TC reads the provided topology file line by line and makes sure that each host has a matching entry in the definitions file which specifies which kind of device it is (faked, switch or server).
2. All devices of one kind, servers or switches, get the same box file (aka version of code) as a starting point which is prescribed in the definition file.
3. Interface remapping files are created in the "./helper_scripts/autogenerated" directory as needed for each VX device. (Remapping is required to make use of custom non-sequential interface names as specified in the topology.dot file.)
4. Vagrant config for each VM is generated and outputted into the Vagrantfile.


# Additional Info: 
* Can load switches and servers with specific configuration for each device type by modifying the "./helper_scripts/extra_[switch|server]_config.sh" file
* 1st and 2nd Device boot order can be specified in the definitions file with special config provided for each in the "./helper_scripts/[1st|2nd]_device_config.sh" file
* Files contained in the "helper_scripts/autogenerated" directory are only required to support interface remapping on Vx (and debian-based host) VMs.
* Vagrantfiles written for the libvirt provider will come up in parallel regardless of the order specified in the Vagrantfile this give libvirt an obvious advantage for simulations with many nodes.


# Caveats
* The Libvirt implementation in use does not support point-to-multipoint interfaces (like Virtualbox). If you need to connect one port to many other ports such as a MGMT node it is recommended to create another VX device to use an OOB switch.
* The Virtualbox provider supports a maximum of 36 interfaces of which one is consumed by the vagrant interface giving an end-user 35 interfaces to interconnect in the topology. (I am not aware of any such interface limitation on libvirt)

# Improvements to make it better: 
* Add Sanity checking for "good" hostnames (no leading numbers, no spaces, no "_" etc)
* Use proper dot file parsing python library