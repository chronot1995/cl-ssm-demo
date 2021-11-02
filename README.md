# Source Specific Multicast Demo

Source Specific Multicast Demo

### Description

This will demo Source Specific Multicast (SSM) on Cumulus Linux

1. First, create a "Cumulus In the Cloud" Reference Topology within Cumulus Air if you're building this demo from scratch. We will be presenting this demo using a subset of the overall cldemo2 topology.

2. Next, copy the gitlab repo unto the oob-mgmt-server:

   ```
   git clone https://gitlab.com/nvidia-networking/systems-engineering/poc-support/source-specific-multicast-demo
   ```

3. Change directories to the following

   ```
   cd source-specific-multicast-demo
   ```

4. Run the following:

   ```
   ansible-playbook ssm.yml
   ```

This will setup a BGP underlay between the leaf01-04 and spine01-02. The remaining links in the cldemo2 topology will be disabled to avoid any issues. This will also setup and configure server01 and server05 for SSM iperf traffic.
<!-- AIR:tour -->
### Testing SSM

1. First, you will need to open a few SSH sessions within Nvidia Air.

2. Log into server05 and start the ssmping server with the following command:

```
ssmpingd &
```

3. Log into server01 and start the ssmping client with the following command:

```
ssmping -4 -I eth1 -c 100 192.168.55.222
```

4. ssmpingd server output:

```
cumulus@server05:~$ ssmpingd
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
received request from 192.168.11.111
```

5. ssmping client output:

```
cumulus@server01:~$ ssmping -4 -I eth1 -c 100 192.168.55.222
ssmping joined (S,G) = (192.168.55.222,232.43.211.234)
pinging S from 192.168.11.111
  unicast from 192.168.55.222, seq=1 dist=3 time=3.406 ms
  unicast from 192.168.55.222, seq=2 dist=3 time=2.914 ms
multicast from 192.168.55.222, seq=2 dist=3 time=3.450 ms
  unicast from 192.168.55.222, seq=3 dist=3 time=4.207 ms
multicast from 192.168.55.222, seq=3 dist=3 time=4.236 ms
  unicast from 192.168.55.222, seq=4 dist=3 time=3.234 ms
multicast from 192.168.55.222, seq=4 dist=3 time=3.576 ms
  unicast from 192.168.55.222, seq=5 dist=3 time=3.556 ms
multicast from 192.168.55.222, seq=5 dist=3 time=3.578 ms
  unicast from 192.168.55.222, seq=6 dist=3 time=3.377 ms
multicast from 192.168.55.222, seq=6 dist=3 time=3.439 ms
  unicast from 192.168.55.222, seq=7 dist=3 time=3.000 ms
multicast from 192.168.55.222, seq=7 dist=3 time=3.059 ms
  unicast from 192.168.55.222, seq=8 dist=3 time=3.141 ms
multicast from 192.168.55.222, seq=8 dist=3 time=3.168 ms
  unicast from 192.168.55.222, seq=9 dist=3 time=3.657 ms
multicast from 192.168.55.222, seq=9 dist=3 time=3.819 ms
  unicast from 192.168.55.222, seq=10 dist=3 time=2.949 ms
```

### PIM SSM Verification and Troubleshooting:

First, let's take a look at the IGMP information on the receiver end / leaf01

```
cumulus@leaf01:mgmt:~$ net show igmp sources
Interface        Address         Group           Source          Timer Fwd Uptime
swp1             192.168.11.1    232.43.211.234  192.168.55.222  03:40   Y 00:00:39
```

In the above example we see the Multicast group and the source of the SSM traffic.

The SSM Multicast stream in our non-ECMP configuration (for simplicity) will be going over one of the two spine connections. Currently, the traffic is traversing spine02.

On spine02, we have a connection down to leaf01 (receiver switch) through swp1:

```
cumulus@spine02:mgmt:~$ net show interface | grep -i "swp1 "
UP     swp1    1G   9216   Default   leaf01 (swp52)
```

and a connection down to leaf04 (sender switch) through swp4:

```
cumulus@spine02:mgmt:~$ net show interface | grep -i "swp4 "
UP     swp4    1G   9216   Default   leaf04 (swp52)
```

We can see the state that is being reported by PIM and that a Join is being requested from leaf01:

```
cumulus@spine02:mgmt:~$ net show pim state
Codes: J -> Pim Join, I -> IGMP Report, S -> Source, * -> Inherited from (*,G), V -> VxLAN, M -> Muted
Active Source           Group            RPT  IIF               OIL
1      192.168.55.222   232.43.211.234   n    swp4              swp1( J   )
```

```
cumulus@spine02:mgmt:~$ net show pim join
Interface        Address         Source          Group           State      Uptime   Expire Prune
swp1             10.1.1.2        192.168.55.222  232.43.211.234  JOIN       00:00:30 03:05  --:--
```

Next, we can see the "upstream" of the pim source, which is the sending switch / leaf04 through swp4:

```
cumulus@spine02:mgmt:~$ net show pim upstream
Iif             Source          Group           State       Uptime   JoinTimer RSTimer   KATimer   RefCnt
swp4            192.168.55.222  232.43.211.234  J           00:00:54 00:00:39  --:--:--  00:03:05       2
```

Finally, we can see PIM is in Sparse Mode and that a tree is being built through swp4 to swp1:

```
cumulus@spine02:mgmt:~$ net show mroute
IP Multicast Routing Table
Flags: S - Sparse, C - Connected, P - Pruned
       R - RP-bit set, F - Register flag, T - SPT-bit set

Source          Group           Flags   Proto  Input            Output           TTL  Uptime
192.168.55.222  232.43.211.234  ST              PIM    swp4             swp1             1    00:01:11
```
<!-- AIR:tour -->
