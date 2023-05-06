There are multiple reasons why you might want to test the speed of you local network. For example you can test the speed between your router and your device. Or the speed between your device and a home server. For me it was the former. I wanted to make sure the connection from my router to my desktop PC was working fine so that I could make the most of the broadband speed I was paying for.

We can use a tool called [IPerf3](https://iperf.fr/). This is available for many platforms including: Windows, Linux, Mac and Android. I installed it on my PC, but I cannot run it on my router, so I installed it on my laptop and connected it directly to another ethernet port on the router.

On one device run the server:

```shell
iperf3 -s
```

On the other device run the client pointing to the IP of the first device to start the test:

```shell
iperf3 -c 192.168.0.123
```

You'll get some results that look like this:

```plain
❯ .\iperf3.exe -c localhost
Connecting to host localhost, port 5201
[  4] local ::1 port 53076 connected to ::1 port 5201
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-1.00   sec  2.08 GBytes  17.8 Gbits/sec
[  4]   1.00-2.00   sec  2.02 GBytes  17.2 Gbits/sec
[  4]   2.00-3.00   sec  2.06 GBytes  17.8 Gbits/sec
[  4]   3.00-4.00   sec  2.53 GBytes  21.7 Gbits/sec
[  4]   4.00-5.00   sec  2.48 GBytes  21.3 Gbits/sec
[  4]   5.00-6.00   sec  2.43 GBytes  20.8 Gbits/sec
[  4]   6.00-7.00   sec  2.52 GBytes  21.6 Gbits/sec
[  4]   7.00-8.00   sec  2.11 GBytes  18.1 Gbits/sec
[  4]   8.00-9.00   sec  2.10 GBytes  18.1 Gbits/sec
[  4]   9.00-10.00  sec  2.57 GBytes  22.1 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-10.00  sec  22.9 GBytes  19.7 Gbits/sec                  sender
[  4]   0.00-10.00  sec  22.9 GBytes  19.7 Gbits/sec                  receiver

iperf Done.
```

Here the average speed was `19.7 Gbits/sec`.

(Note that this was unrealistically high, because in this case, I ran it against `localhost`, i.e. the same device.)
