# Source-Specific Multicast Demo

Source-Specific Multicast Demo

### Description

This will demo Source-Specific Multicast (SSM) on Cumulus Linux

1. First, create a "Cumulus In the Cloud" Reference Topology within Cumulus Air. We will be presenting this demo using a subset of the overall cldemo2 topology.

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

### Setting up

1. First, you will need to open a few SSH sessions within Cumulus Air.

2. Log into server05 and start the iperf server in SSM mode with the following command:

```
iperf -s -u -B 239.5.5.5 -i 1
```

3. Log into server01 and start the iperf client in SSM mode with the following command:

```
iperf -c 239.5.5.5 -u -T 32 -t 500 -i 1
```

4. iperf server output:

```
cumulus@server05:~$ iperf -s -u -B 239.5.5.5 -i 1
------------------------------------------------------------
Server listening on UDP port 5001
Binding to local address 239.5.5.5
Joining multicast group  239.5.5.5
Receiving 1470 byte datagrams
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 239.5.5.5 port 5001 connected with 192.168.200.31 port 50928
[ ID] Interval       Transfer     Bandwidth        Jitter   Lost/Total Datagrams
[  3]  0.0- 1.0 sec   129 KBytes  1.06 Mbits/sec   0.246 ms    0/   90 (0%)
[  3]  1.0- 2.0 sec   128 KBytes  1.05 Mbits/sec   0.674 ms    0/   89 (0%)
[  3]  2.0- 3.0 sec   128 KBytes  1.05 Mbits/sec   0.290 ms    0/   89 (0%)
[  3]  3.0- 4.0 sec   128 KBytes  1.05 Mbits/sec   0.215 ms    0/   89 (0%)
[  3]  4.0- 5.0 sec   128 KBytes  1.05 Mbits/sec   0.251 ms    0/   89 (0%)
[  3]  5.0- 6.0 sec   128 KBytes  1.05 Mbits/sec   0.204 ms    0/   89 (0%)
[  3]  6.0- 7.0 sec   129 KBytes  1.06 Mbits/sec   0.229 ms    0/   90 (0%)
[  3]  7.0- 8.0 sec   128 KBytes  1.05 Mbits/sec   0.294 ms    0/   89 (0%)
[  3]  8.0- 9.0 sec   128 KBytes  1.05 Mbits/sec   0.272 ms    0/   89 (0%)
[  3]  9.0-10.0 sec   128 KBytes  1.05 Mbits/sec   0.248 ms    0/   89 (0%)
[  3] 10.0-11.0 sec   128 KBytes  1.05 Mbits/sec   0.252 ms    0/   89 (0%)
[  3] 11.0-12.0 sec   128 KBytes  1.05 Mbits/sec   0.300 ms    0/   89 (0%)
```

5. iperf client output:

```
cumulus@server01:~$ iperf -c 239.5.5.5 -u -T 32 -t 500 -i 1
------------------------------------------------------------
Client connecting to 239.5.5.5, UDP port 5001
Sending 1470 byte datagrams, IPG target: 11215.21 us (kalman adjust)
Setting multicast TTL to 32
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 192.168.200.31 port 50928 connected with 239.5.5.5 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 1.0 sec   131 KBytes  1.07 Mbits/sec
[  3]  1.0- 2.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  2.0- 3.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  3.0- 4.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  4.0- 5.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  5.0- 6.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  6.0- 7.0 sec   129 KBytes  1.06 Mbits/sec
[  3]  7.0- 8.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  8.0- 9.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  9.0-10.0 sec   128 KBytes  1.05 Mbits/sec
[  3] 10.0-11.0 sec   128 KBytes  1.05 Mbits/sec
[  3] 11.0-12.0 sec   128 KBytes  1.05 Mbits/sec
```

### PIM SSM Verification:
