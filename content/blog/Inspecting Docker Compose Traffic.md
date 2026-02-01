+++
title= "Inspecting Docker Compose Traffic"
date= "2026-02-01"
tags= ["post"]
draft= true
+++

This assumes you have a running docker setup with a named bridge network

1. Use `bridge link` to view bridge interfaces

2. if you run ip link or ip a in your container, it will show the veth as something like eth0@ifXX, where XX is a number. If you then run ip link (ip a) for the host, the indices at the left of the interface names will correspond to those XX values---so whichever interface has that index is the host end of your container veth

3. wait, it will work. run this on host: `for ns in $(ip netns | cut -f 1 -d ' '); do echo $ns; sudo ip netns exec $ns ip link; done` though you'll need to figure out which netns corresponds to your container

4. network ns is found in `docker inspect [name] | grep SandboxKey`

5. Script to list all eth0 interfaces on running docker containers:

```

#!/bin/bash

for container in $(docker ps -q); do

    name=`docker ps | grep $container | awk -F " " '{ print $2 }'`

    iflink=`docker exec $container cat /sys/class/net/eth0/iflink`

    iflink=`echo $iflink|tr -d '\r'`

    veth=`grep -l $iflink /sys/class/net/veth*/ifindex`

    veth=`echo $veth|sed -e 's;^.*net/\(.*\)/ifindex$;\1;'`

    echo $name:$veth

done

```

1. Use `tcpdump -i <interface> -w output.cap`

2. Extra Credit: You can run multiple instances of tcpdump on different interfaces (using background tasking or multiple terminal windows) and then merge them together using wireshark's `mergecap -w merged.cap output1.cp output2.cap output3.cap`

3. When done capturing, end the tcpdump command

4. Load the cap file into a viewer

5. CLI: `tcpick -C -yP -r output.cap

6. GUI: Load it into Wireshark

7. Alias the IPs for easier reading

8. Use `docker ps` to get the running container ids

9. Use `docker inspect` to get the IPs of the container/their network interface

10. In Wireshark enable View -> Name Resolution -> Resolve Network Addresses

11. Right click on an IP and click "Edit Resolved Name"

12. Type the name of the container with that IP