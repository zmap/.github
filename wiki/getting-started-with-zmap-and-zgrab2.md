# Getting Started with ZMap and ZGrab2

## Welcome

We've found users commonly use both [ZMap](https://github.com/zmap/zmap) and [ZGrab2](https://github.com/zmap/zgrab2) together in a measurement pipeline for efficient port probing and protocol scanning.
Services on the internet are often quite ephemeral and ensureing ZGrab2 scans immediately after ZMap helps ensure we scan services while they're still up.

- ZMap is a fast L4 scanner that by default sends out TCP SYN packets to elicit a response from internet hosts. It can scan the entire IPv4 space in 45 mins with 1 Gb/s links.
- ZGrab2 is a modular application-layer scanner with support for common protocols like `ssh`, `http`, and many others.

We have Getting Started guides for [ZMap](https://github.com/zmap/zmap/wiki/Getting-Started-Guide) with a bit more details about ZMap specficially.
This guide is for people brand new to internet measurement who want to use these tools in a pipeline or who'd like some pointers to helpful CLI utilities and techniques.

> [!NOTE]
>I'd encourage you to read this document in full as it includes helpful insights and motivations gained through using the tools for some time that will help you make more informed decisions while scanning.
>
>That being said, for a comprehensive example head to [Putting it all Together](#putting-it-all-together).

> [!CAUTION]
> Without care, internet scanning can cause a Denial-of-Service attack on a network. Ethical scanning is important for both altruism and for measurement result accuracy. Read more in our [Ethical Scanning section](#ethical-scanning---avoiding-causing-a-denial-of-service).

## Installation

While we endeavor to keep package managers up-to-date, the fixed release schedule of package managers like `apt` means that depending on your OS and OS version, that version could be quite out-of-date.

Installing from source is easy and guarantees you have the most recent version.
The below instructions are copied for your conveinence, if you need to install for an OS other than Ubuntu/MacOS, follow the full instructions for [ZMap](https://github.com/zmap/zmap/blob/main/INSTALL.md#building-from-source)and [ZGrab2](https://github.com/zmap/zgrab2?tab=readme-ov-file#building-from-source).
The below sequence will install a `zmap` and `zgrab2` binary so it will be accessible system-wide.

### To install ZMap on Ubuntu/Mac

1. Install the build dependencies as described with the below command depending on your OS:
    a. On Ubuntu

    ```shell
    sudo apt-get install build-essential cmake libgmp3-dev gengetopt libpcap-dev flex byacc libjson-c-dev pkg-config libunistring-dev libjudy-dev
    ```

    b. On MacOS with [homebrew](https://brew.sh)

    ```shell
    brew install pkg-config cmake gmp gengetopt json-c byacc libunistring judy
    ```

1. Run the below command to clone the repository, build, and install ZMap:

    ```shell
    git clone https://github.com/zmap/zmap.git /tmp/zmap;
    cd /tmp/zmap;
    cmake .; make -j 4; sudo make install;
    cd ~/
    ```

1. Close and re-open your terminal and test that you've installed correctly

    ```shell
    zmap --version
    ```

    If installed correctly, you should see a version like:

    ```shell
    zmap v4.3.4
    ```

### To install ZGrab2 on Ubuntu/Mac

0. Install Golang following their installation [instructions](https://go.dev/doc/install) if you don't have Go already
1. Run the following:

    ```shell
    git clone https://github.com/zmap/zgrab2.git /tmp/zgrab2;
    cd /tmp/zgrab2;
    sudo make install
    ```

2. Close your current terminal, re-open terminal, and test that you've installed correctly

```shell
zgrab2 --help
```

If installed correctly, you should see the help page

```shell
Usage:
  zgrab2 [OPTIONS] <command>
...
```

## Running your first scan

If everything is installed correctly, let's dive right in to doing a sample scan for `ssh` services.

### ZMap - Open Port Scanner

Run the following:

```shell
sudo zmap -p 22 --bandwidth 7M --max-results 10
```

You should see something similar to:

```txt
 0:00 0%; send: 1 1 p/s (77 p/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
185.39.222.38
45.77.183.125
46.13.197.200
152.70.181.175
34.228.77.22
103.107.52.128
89.229.58.117
120.27.212.209
91.98.195.120
197.15.208.221
 0:01 12%; send: 3693 done (11.2 Kp/s avg); recv: 10 10 p/s (9 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.27%
```

This is sending TCP SYN (the first packet of TCP's 3-way handshake) probes to port 22 (`ssh`'s traditional port assignment) at a rate of 7 Mbit/s and we stop scanning after receiving a response from 10 services.

You won't see those exact IPs since both:

1. Services on the internet are constantly going on and off-line, so what you see today will be different than what you see tomorrow
2. ZMap randomizes the order in which it scans IPs by default, so each invocation will scan the IPv4 space with a different ordering.

At this point, we don't *know* that those 10 IP addresses are running `ssh`, we just know that something responded to our TCP handshake. That's where `zgrab2` comes in.

### ZGrab2 - Application Protocol Handshake

Take one of your IPs from ZMap and run the following:

```shell
echo "103.107.52.128" | zgrab2 ssh
```

If there's an active `ssh` server there, you'll see a fields like the SSH server version, the key exchange algorithms and cipher suites the server offered, and the server's public key.

```shell
INFO[0000] started grab at 2026-04-11T00:24:10Z
{"ip":"103.107.52.128","data":{"ssh":{"status":"success","protocol":"ssh","port":22,"result":{"server_id":{"raw":"SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.13","version":"2.0",...
INFO[0000] finished grab at 2026-04-11T00:24:10Z
00h:00m:00s; Scan Complete; 1 targets scanned; 1.18 targets/sec; 100.0% success rate
{"statuses":{"ssh":{"successes":1,"failures":0}},"start":"2026-04-11T00:24:10Z","end":"2026-04-11T00:24:10Z","duration":"850.010417ms","zgrab_cli_parameters":"zgrab2 ssh","num_targets_scanned":1}
```

If there is no `ssh` server running on that IP, you'll see this instead:

```shell
{"ip":"216.254.5.12","data":{"ssh":{"status":"handshake-error","protocol":"ssh","port":22,"timestamp":"2026-04-11T00:27:57Z","error":"failed to create SSH client connection: ssh: handshake failed: EOF"}}}
INFO[0000] finished grab at 2026-04-11T00:27:57Z
00h:00m:00s; Scan Complete; 1 targets scanned; 42.27 targets/sec; 0.0% success rate
{"statuses":{"ssh":{"successes":0,"failures":1}},"start":"2026-04-11T00:27:57Z","end":"2026-04-11T00:27:57Z","duration":"23.580461ms","zgrab_cli_parameters":"zgrab2 ssh","num_targets_scanned":1}
```

This is why we need to use ZMap and ZGrab2 in tandem, ZMap identifies *possible services* and ZGrab2 *confirms* a service is running a specific application protocol.

## Running ZMap and ZGrab2 in a pipeline

While the above workflow works fine for single IPs, you'll often to use them in tandem.

ZMap by default outputs IPs to `stdout`, which we can pipe directly to ZGrab2's `stdin`. Zgrab2 with then output final `ssh` scan results to `stdout` where it'll show up in your terminal.

Run:

```shell
sudo zmap -p 22 --bandwidth 7M --max-results 10 | zgrab2 ssh
```

Each output line is JSON.

```text
 0:00 0%; send: 1 1 p/s (75 p/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
{"ip":"208.94.216.126","data":{"ssh":{"status":"handshake-error","protocol":"ssh","port":22,"timestamp":"2026-04-11T00:3
3:31Z","error":"failed to create SSH client connection: ssh: handshake failed: read tcp 171.67.71.209:44004-\u003e208.94
.216.126:22: read: connection reset by peer"}}}
{"ip":"38.65.90.220","data":{"ssh":{"status":"success",...
```

Great! Now we can scan for `ssh` servers on port 22.

### Getting N Successful ZGrab2 Scans

You may want to get a set number of successful ZGrab2 scan results.
For example, you may want 100 successful SSH handshakes with random hosts as a sample size to analyze SSH server versions on the internet.

Using `jq`, we can filter for only the ZGrab2 results that ended in a successful result and use `head` to limit the output (and killing the entire pipeline) when we've reached 100 lines.
Run:

```shell
sudo zmap -p 22 -B 500M | zgrab2 ssh | jq -c 'select(.data[].status == "success")' | head -n 100 > /tmp/100-successful-ssh-handshakes
```

Now we have 100 successful responses saved to a file. View the last one with:

```shell
tail -n 1 /tmp/100-successful-ssh-handshakes | jq 
```

## Ethical Scanning - Avoiding causing a Denial-of-Service

Ethical scanning is important both for other internet users and the accuracy of your scan.

Scanning a small enough network at a high enough speed can at best make your scanning very apparent to anyone looking (and people do watch this stuff, I get emails from people whenever someone in our lab scans too aggressively over a certain corner of the internet) and at worst can cause network infrastructure to fail.

Besides causing pain to others, if people notice your scanning they can just block incoming traffic from your network. This means your scans will not pick up anything within their network from then on, decreasing your accuracy.

This brings up 2 guiding principles

1. Always scan at the slowest rate possible given your research question
2. The smaller the subnet or list of targets your scanning, the slower you should scan.

### Smaller the target -> slower the rate

While it's difficult to have rules of scanning speed that are always applicable (some networks can handle more load, some may be pushed over more easily by scanning), 1 Gb/s will be perfectly fine for the entire IPv4 space and a scan should complete in 45 minutes, but you would **not** want to scan a /24 (256 hosts) **anywhere** near 1 Gb/s.

The smaller the target space, the more load any given router will see based on your scanning.

![target_subnet_load](https://github.com/zmap/zmap/assets/23459798/8642008a-df16-40ac-a7ec-d9389964e7b3)

At 1 Gb/s, a single IP would see a probe once in every 45 minutes,  a /16 would see 24 packets per second (imagine a router that serves a /16, it has to forward all ZMap probes to every host behind it), and a /24 subnet would see a packet every 10 seconds.

As rules of thumb, here are the corresponding packets per second for smaller subnets such that any given subnet would see the same load as a full IPv4 scan at 1 Gb/s:

| Subnet Size                   | Bandwidth (--bandwidth) | Scan Rate in pps (--rate)           |
| ----------------------------- | ----------------------- | ----------------------------------- |
| 0.0.0.0/0 (Entire IPv4 Space) | 1 Gb/s                  |                                     |
| 0.0.0.0/8                     | 3.9                     |                                     |
| 0.0.0.0/16                    |                         |                                     |
| 0.0.0.0/24                    |                         |                                     |

10 Gb/s is likely the upper limit of what you'd want to scan the entire IPv4 space, a /16 router would see 220 packets per second and a /24 router would see ~1 packet per second. I personally take 1 packet per second per /24 subnet as a hard upper bound.

### Example - /16

If you want to scan a /16 so it will see the same rate of incoming packets as it would at 10 Gb/s IPv4-wide scan, use:

```shell
zmap -p 22 --bandwidth 153K 171.67.71.1/16
```

### Example - /24

If you want to scan a /24 so it will see the same rate of incoming packets as it would at 10 Gb/s IPv4-wide scan, use:

```shell
zmap -p 22 --rate 1 171.67.71.1/24
```

## Differing Scan Rates between ZMap and ZGrab2

If you run a full IPv4 scan at 1 Gb/s with ZMap and pipe results into ZGrab2, you'll quickly see ZMap begins to "drop" responses.

Here we'll direct ZGrab2's output to `/dev/null` to drop it since we don't care what the results are and it'll clutter out output.

Run:

```shell
sudo zmap -p 22 -B 1G | zgrab2 ssh > /dev/null
```

Gives:

```shell
INFO[0000] started grab at 2026-04-11T01:49:23Z
Apr 11 01:49:23.862 [INFO] recv: duplicate responses will be excluded from output
Apr 11 01:49:23.862 [INFO] recv: unsuccessful responses will be excluded from output
 0:00 0%; send: 1095 1 p/s (62.9 Kp/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
00h:00m:01s; 1313 targets scanned; 1312.85 targets/sec; 85.8% success rate
 0:01 0%; send: 1467693 1.47 Mp/s (1.44 Mp/s avg); recv: 8695 8.69 Kp/s (8.54 Kp/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.59%
00h:00m:02s; 2759 targets scanned; 1379.42 targets/sec; 88.3% success rate
Apr 11 01:49:25.867 [WARN] monitor: Dropped 27008 packets in the last second, (27013 total dropped (pcap: 27013 + iface: 0))
 0:02 0%; send: 2947794 1.48 Mp/s (1.46 Mp/s avg); recv: 12464 3.77 Kp/s (6.18 Kp/s avg); drops: 27.0 Kp/s (13.4 Kp/s avg); hitrate: 0.42%
00h:00m:03s; 4025 targets scanned; 1341.62 targets/sec; 89.3% success rate
Apr 11 01:49:26.867 [WARN] monitor: Dropped 50564 packets in the last second, (77593 total dropped (pcap: 77593 + iface: 0))
```

ZMap and ZGrab2 both output per-second update messages on the progress of the scan. We can see that ZMap is sending at a rate of ~1.44 M packets per second and is receiving ~8K pps responses here:

```shell
 0:01 0%; send: 1467693 1.47 Mp/s (1.44 Mp/s avg); recv: 8695 8.69 Kp/s (8.54 Kp/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.59%
```

Each of the 8k responses per second are being fed into ZGrab2, but it can't scan anywhere near that rate. It's scanning at 1,341 per second.

```shell
00h:00m:03s; 4025 targets scanned; 1341.62 targets/sec; 89.3% success rate
```

This mismatch in ZMap's receiving rate and ZGrab2's scanning rate is causing the `WARN` messages about ZMap dropping packets:

```shell
Apr 11 01:49:25.867 [WARN] monitor: Dropped 27008 packets in the last second, (27013 total dropped (pcap: 27013 + iface: 0))
```

ZGrab2 can't scan as fast as ZMap is outputting targets which leads to the pipe's (`|`) finite buffer filling up and then blocking ZMap from outputting received responses.

While frustrating, this is a consequence of ZMap's asynchronous sending and receiving behavior that makes it so fast: when the receiving thread can't output a received response it can't notify the sending thread to send slower so it just has to drop some of the responses.

### Using a file to decouple ZMap/ZGrab2

One possible solution is to write all ZMap hits to a file for later scanning with ZGrab2.

Run:

```shell
sudo zmap -p 22 -B 1G --max-results 10000 > /tmp/zmap-hits
```

and then after ZMap completes

```shell
cat /tmp/zmap-hits | zgrab2 ssh > /tmp/zgrab2-ssh-scan-results
```

Decoupling ZMap and ZGrab2 runs works great, but it introduces delay between when an IP is seen by ZMap and when it's scanned with ZGrab2.

This delay can cause hosts that were hit successfuly with ZMap go offline by the time they're scanned with ZGrab2.

### Reducing the delay between ZMap and ZGrab2

We'd love to be able to scan a ZMap hit with ZGrab2 as soon as possible after scanning so using a file to decouple isn't ideal.

We need to do two things to fully handle this:

1. Better match the receive rate of ZMap to the send rate of ZGrab2
2. Introduce a dynamic buffer between ZMap and ZGrab2 that can accept the flux and buffer the output of ZMap and input of ZGrab2

Any buffer introduces delay (in a way, the file used above was a type of buffer), but by closer matching the rates in #1, we should be able to minimize this delay.

#### `ztee`

`ztee` was installed alongside `zmap`, and serves as a dynamic buffer to handle our #2 need.

Run:

```shell
sudo zmap -p 22 -B 1G | ztee --raw /dev/null | zgrab2 ssh --target-timeout=30s > /dev/null
```

We no longer see the `[WARN] monitor: Dropped 50564 packets` warning, yay!

`ztee` is a dynamically sized buffer and can increase in size as needed to hold ZMap's output.

However, the mis-match of send/recv rates means this buffer will continue to grow, eating up memory (run the above with `htop` and you'll see!) and adding more and more delay between when `zmap` sends a probe and when `zgrab2` scans that target.

#### Matching Rates

Lowering the rate of ZMap will mean the ZMap scan takes longer, but if you're going to pipe the results into ZGrab2 anyway, so ZGrab2 is the limiting factor.

ZGrab2's delay depends on the module (some modules require more round trips during the protocol), but let's start by reducing the bandwidth to `-B 35M` and using `ztee`.

Run:

```shell
sudo zmap -p 22 -B 35M | ztee --raw /dev/null | zgrab2 ssh --target-timeout=30s > /tmp/zgrab-scan
```

On my machine (behavior could depend on your vantage point and network), this results in `zmap`'s received rate being just slightly higher than `zgrab2`'s scan rate.

I've annotate the lines for clarity:

```shell
zgrab2 - 00h:02m:26s; 53357 targets scanned; 365.46 targets/sec; 83.1% success rate
zmap - 2:26 0% (19h left); send: 7589546 52.1 Kp/s (52.0 Kp/s avg); recv: 54114 365 p/s (370 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.71%
```

`zmap` is receiving 370 packets per second on average and `zgrab2` is scanning at a rate of 365/sec.

With `ztee` in the middle buffering the two, we no longer see any errors, the memory usage isn't growing, and there will be minimal delay between when ZMap receives a packet and ZGrab2 scans it.

#### ZGrab2 Performance

Three knobs with ZGrab2 typically affect performance.

- `--senders` - the number of sending threads used
- `--connect-timeout` - how long a ZGrab2 sender will wait for the initial connection to be established. For TCP this is a 3-way handshake.
- `--target-timeout` - how logn ZGrab2 will wait for an entire protocol scan of a single target to take

ZGrab2 will spawn N sender threads with each thread completing a full protocol scan of the target synchronously. Once complete, the sender thread will grab a new target to scan.

The default values of these arguments are set to optimize for accuracy *potentially* at the cost of performance.

You can increase the number of senders and decrease the timeouts so that senders aren't blocked on non-responsive hosts for as long.

> [!TIP]
> You'll quickly run into a case of diminishing returns as well as potentially a reduction in accuracy. Some target subnets will more readily recognize your scanning as coming from a scanner rather than legitimate traffic and can throttle/block you should you increase your scan rate. These will usually show up in the scan as an increase in timeouts. The default values are set to be a good compromise between performance and timeouts but feel free to experiment with a configuration that works well for you and what you're interested in.

## Putting it all Together

Here is an example pipeline to get N successful `ssh` targets using both ZMap and ZGrab2 with all options previously discussed and placing the results in a file.

```shell
sudo zmap -p 22 --bandwidth 50M | ztee --raw /dev/null | zgrab2 ssh --senders=1000 --connect-timeout=10s --target-timeout=30s | jq -c 'select(.data[].status == "success")' | head -n 100 > /tmp/100-successful-ssh-handshakes
```

`head` will terminate the program after 100 responses are collected, so you can disregard `ztee` complaining it can't finish writing.

```shell
Apr 21 21:16:58.694 [FATAL] ztee: Error writing to stdout
```

You now have 100 SSH successful scan results collected. Use `[zmap/zgrab2] --help` to learn more about what each command line flag does.

## Improving this Guide

If you're getting started with these tools and have questions not answered here, please open an issue and explain what you're stuck on so we can both help you and improve our documentation.