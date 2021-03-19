## Troubleshooting Guide:
This guide provides a searchable list of errors for a possible match/solution.


### 1. Missing `dg_name` in `cluster_config.json` manifestation:
* If this is the error text:

```
fatal: [linuxserver44.orcl]: FAILED! => {"msg": "'ansible.vars.hostvars.HostVarsVars object' has no attribute 'dg_name'"}
```

* then this could possibly be the place to check:

```
✔ ~/git-clone-holder/bms-toolkit [master L|✚ 3…3] 
00:52 $ cat cluster_config.json 
[
  {
    "scan_name": "dec-test-scan-cluster",
    "scan_port": "1521",
    "cluster_name": "dec-test-cluster",
    "cluster_domain": "dec-test-home",
    "public_net": "dec-test-public_interface",
    "private_net": "dec-test-private",
    "scan_ip1": "172.16.30.20",
    "scan_ip2": "172.16.30.21",
    "scan_ip3": "172.16.30.22",
    "dg_name": "DATA",<======================================== this parameter was missing in this file's this line
    "nodes": [
      {  "node_name": "linuxserver44.orcl",
         "host_ip": "172.16.30.1",
         "vip_name": "linuxserver44-vip.orcl",
         "vip_ip": "172.16.30.11"
      },
      {  "node_name": "linuxserver45.orcl",
         "host_ip": "172.16.30.2",
         "vip_name": "linuxserver45-vip.orcl",
         "vip_ip": "172.16.30.12"
      }
    ]
  }
]

```

### 2. Error manifestation with respect to an invalid disk group name:
* For following error:

```
TASK [rac-gi-setup : rac-gi-install | Get symlinks for devices] ************************************************************************************************
task path: /home/jcnarasimhan/git-clone-holder/bms-toolkit/roles/rac-gi-setup/tasks/rac-gi-install.yml:127
File lookup using /home/jcnarasimhan/git-clone-holder/bms-toolkit/asm_disk_config.json as file
fatal: [linuxserver44.orcl]: FAILED! => {
    "msg": "Invalid data passed to 'loop', it requires a list, got this instead: . Hint: If you passed a list/dict of just one element, try adding wantlist=True to your lookup invocation or use q/query instead of lookup."
}
```

* Check that: `dg_name=DATA` points to a valid diskgroup name.
* Rationale: the expression:
```
"{{ asm_disks | json_query('[?diskgroup==`' + hostvars[groups['dbasm'].0]['dg_name'] + '`].disks[*].blk_device') | list | join() }}"
```

... translates to something similar to the following:
```
"{{ asm_disks | json_query('[?diskgroup==`'DATA'`].disks[*].blk_device') | list | join() }}"
```

where the 'DATA' is the incorrect DG name supplied in cluster_config.json file's `"dg_name": "DATA"`

### 3. Error manifestation due to an invalid network interface name
* Error:
```
TASK [rac-gi-setup : rac-gi-install | Create GI response file] *************************************************************************************************
fatal: [linuxserver44.orcl]: FAILED! => {"changed": false, "msg": "AnsibleUndefinedVariable: 'dict object' has no attribute u'dec-test-public_interface'"}
```

* Fix:
Ensure that the entries for the n/w interfaces are correctly defined:
```
✔ ~/git-clone-holder/bms-toolkit [master L|✚ 114…15] 
20:02 $ cat cluster_config.json
20:03 $ cat cluster_config.json.enough_fun
[
  {
    "scan_name": "dec-test-scan-cluster",
    "scan_port": "1521",
    "cluster_name": "dec-test-cluster",
    "cluster_domain": "dec-test-home",
    "public_net": "dec-test-public_interface",   <==================== this and 
    "private_net": "dec-test-private",           <==================== this should reflect the actual interfaces on the BM hosts 
    "scan_ip1": "172.16.30.20",
    "scan_ip2": "172.16.30.21",
    "scan_ip3": "172.16.30.22",
    "dg_name": "DATA",
    "nodes": [
      {  "node_name": "linuxserver44.orcl",
         "host_ip": "172.16.30.1",
         "vip_name": "linuxserver44-vip.orcl",
         "vip_ip": "172.16.30.11"
      },
      {  "node_name": "linuxserver45.orcl",
         "host_ip": "172.16.30.2",
         "vip_name": "linuxserver45-vip.orcl",
         "vip_ip": "172.16.30.12"
      }
    ]
  }
]

```

The interface to be used for `public_net` is the interface that carries the network addrees for the `172.16.30.0` (in the example above) network.
* Example:
```
[root@linuxserver44 ~]# ifconfig -a bond0.111
bond0.111: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.30.1  netmask 255.255.255.0  broadcast 172.16.30.255
        inet6 fe80::3680:dff:fe59:7f1e  prefixlen 64  scopeid 0x20<link>
        ether 34:80:0d:59:7f:1e  txqueuelen 1000  (Ethernet)
        RX packets 1277524  bytes 13573540106 (12.6 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1227353  bytes 621025389 (592.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

The interface to be used for `private_net` is the one that carries the IP for the private interconnect which will be used exclusively for RAC traffic, for example `192.168.3.1` (in the example below):

```
[root@linuxserver44 ~]# ifconfig -a bond1.112
bond1.112: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.3.1  netmask 255.255.255.0  broadcast 192.168.3.255
        inet6 fe80::3680:dff:fe59:7f1f  prefixlen 64  scopeid 0x20<link>
        ether 34:80:0d:59:7f:1f  txqueuelen 1000  (Ethernet)
        RX packets 5664  bytes 260540 (254.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5670  bytes 238460 (232.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 4. Error:

```
SEVERE:  [Jan 20, 2021 7:42:09 AM] [FATAL] [INS-30530] Following specified disks have invalid header status: 
[/dev/asmdisks/DATA_7539397476, /dev/asmdisks/DATA_7539397477, /dev/asmdisks/DATA_7539397478, /dev/asmdisks/DATA_7539397531]
```

Fix: Possibly the last time cleanup-oracle.sh was run, it didn't go thru' till the end, as it zeroes out headers cleanly for you.

So, manually following can be run to cleanup as root:
```
for i in /dev/asmdisks/DATA_7539397476, /dev/asmdisks/DATA_7539397477, /dev/asmdisks/DATA_7539397478, /dev/asmdisks/DATA_7539397531 ; \
do echo "checking disk $i";dd if=$i bs=128 count=1 | od -a ;echo; \
done
```

You will see something like this which shows that the disk headers are not empty and it has remnant DG information:
```
checking disk /dev/asmdisks/DATA_7539397476
1+0 records in
1+0 records out
128 bytes (128 B) copied, 0.000275708 s, 464 kB/s
0000000 soh stx soh soh nul nul nul nul nul nul nul nul   L   .  us can
0000020 nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul
0000040   O   R   C   L   D   I   S   K nul nul nul nul nul nul nul nul
0000060 nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul
0000100 nul nul etx dc3 nul nul soh etx   D   A   T   A   _   0   0   0
0000120   0 nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul
0000140 nul nul nul nul nul nul nul nul   D   A   T   A nul nul nul nul
0000160 nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul nul
0000200
```

Cleanup as:
```
#First backup:
dd if=/dev/asmdisks/DATA_7539397476 of=dev_asmdisks_DATA_7539397476.txt bs=1048576 count=100

#Then cleanup:
for i in /dev/asmdisks/DATA_7539397476 /dev/asmdisks/DATA_7539397477 /dev/asmdisks/DATA_7539397478 /dev/asmdisks/DATA_7539397531; \
do dd if=/dev/zero of=$i bs=1048576 count=100 ;echo Done blowing up header for disk $i; done
```

### 5. Existing ORACLE_HOME

Error:
```
"Launching Oracle Grid Infrastructure Setup Wizard...", "", "[WARNING] [INS-40109] The specified Oracle Base location is not empty on this server.", "   ACTION: Specify an empty location for Oracle Base.", "[WARNING] [INS-32047] The location (/u01/app/oraInventory) specified for the central inventory is not empty.", "   ACTION: It is recommended to provide an empty location for the inventory.", "[FATAL] [INS-13019] Some mandatory prerequisites are not met. These prerequisites cannot be ignored."
```

Fix:
Rerun cleanup-oracle.sh and ensure it finished successfully
