---
title: "Honeypot October 2024 - Using Claude and Pandas"
date: 2024-10-14
tags: ["Honeypot","Claude","Pandas","LLM"]
description: Analyzing a single day of events on my honeypot.
draft: false
toc: true
image: "media/Figure_1.png"
---

## Introduction

The SANS Technology Institute's Internet Storm Center offers a [honeypot service](https://www.dshield.org/tools/honeypot/) that anyone can install to monitor cyber attacks. The honeypot, named DShield Honeypot, sends data to the ISC, and you can create an account to view your own logs and statistics on their website. This past August I set up one of these honeypots on a free AWS instance to see what attacks my instance would get while also gaining experience reading logs and examining external attacks.

## Instance Info

I am running an [Amazon Web Server EC2](https://aws.amazon.com/ec2/) instance. The instance is located in Northern California, and it is running Ubuntu with the DShield Honeypot installed through [this tutorial](https://youtu.be/fMqhoNnyvmE).

## Generating Quick Stats with Claude and Pandas

Now I don't have much experience with data science, and I don't have time to learn it (because I'm writing this blog post the day before I post it), so this seems like a great excuse to use a large language model: my LLM of choice is [Claude](https://www.anthropic.com/claude).

I first give Claude my current scenario: I have some firewall logs that I can save in three formats (XML, JSON, TAB), and I need a tool that will give me some statistics on these logs. Claude offers me some tools, like XMLStarlet and awk, but also some professional software, like Splunk and ELK stack. I clarify with Claude that I am only doing this in my free time, and I need something that is free and easy to use. Claude recommends using Python with pandas. I don't have experience with pandas, but I do know Python so I go with that.

I give Claude two log entries from my honeypot's firewall and I ask it what statistics could I run on it. Claude gives me a Python script that uses pandas, and after some back and forth and some debugging and editing, I get this final script:

```python
import pandas as pd
import matplotlib.pyplot as plt
import json

# Load data from single-line JSON file
with open('firewall_logs.json', 'r') as file:
    data = json.load(file)

# Convert to DataFrame
df = pd.DataFrame(data)

# Combine date and time into a single datetime column
df['datetime'] = pd.to_datetime(df['date'] + ' ' + df['time'], format='%Y-%m-%d %H:%M:%S')

# 1. Count of events by source IP
source_counts = df['source'].value_counts()
print("1. Events by source IP:\n", source_counts.head())

# 2. Count of events by target port
target_port_counts = df['targetport'].value_counts()
print("\n2. Events by target port:\n", target_port_counts.head())

# 3. Events over time (hourly bins)
df['hour'] = df['datetime'].dt.floor('H')
hourly_counts = df['hour'].value_counts().sort_index()
print("\n3. Events per hour:\n", hourly_counts)

# 4. Protocol distribution
protocol_counts = df['protocol'].value_counts()
print("\n4. Protocol distribution:\n", protocol_counts)

# 5. Top 5 source ports
top_source_ports = df['sourceport'].value_counts().head()
print("\n5. Top 5 source ports:\n", top_source_ports)

# 6. Calculate time differences between events
df = df.sort_values('datetime')
df['time_diff'] = df['datetime'].diff().dt.total_seconds()
avg_time_between_events = df['time_diff'].mean()
print(f"\n6. Average time between events: {avg_time_between_events:.2f} seconds")

# Visualization: Events over time
plt.figure(figsize=(12, 6))
hourly_counts.plot(kind='bar')
plt.title('Events Over Time')
plt.xlabel('Hour')
plt.ylabel('Number of Events')
plt.tight_layout()
plt.show()

# Visualization: Protocol Distribution
plt.figure(figsize=(8, 8))
protocol_counts.plot(kind='pie', autopct='%1.1f%%')
plt.title('Protocol Distribution')
plt.ylabel('')
plt.show()
```

Examining my firewall log history on dshield.org, I see a huge report spike on September 20th, 2024. I download the reports for that day, and I plug it into the script.

Here is the output:

```console
$ python3 firewall-log-analyzer.py
1. Events by source IP:
 source
172.233.32.48      32612
172.233.56.83      18280
172.233.56.7       17024
172.235.166.121     6276
165.154.0.174       4304
Name: count, dtype: int64

2. Events by target port:
 targetport
23      77945
2222     1410
443       910
80        797
8728      552
Name: count, dtype: int64

3. Events per hour:
 hour
2024-09-20 00:00:00     6044
2024-09-20 01:00:00     1914
2024-09-20 02:00:00     1712
2024-09-20 03:00:00     1584
2024-09-20 04:00:00     1514
2024-09-20 05:00:00     1488
2024-09-20 06:00:00     1658
2024-09-20 07:00:00     1558
2024-09-20 08:00:00     1682
2024-09-20 09:00:00     1632
2024-09-20 10:00:00     1582
2024-09-20 11:00:00     1074
2024-09-20 12:00:00     1196
2024-09-20 13:00:00    77642
2024-09-20 14:00:00     1316
2024-09-20 15:00:00     1086
2024-09-20 16:00:00     1162
2024-09-20 17:00:00     1364
2024-09-20 18:00:00     1436
2024-09-20 19:00:00     1514
2024-09-20 20:00:00     1494
2024-09-20 21:00:00     2246
2024-09-20 22:00:00     1602
2024-09-20 23:00:00     1368
Name: count, dtype: int64

4. Protocol distribution:
 protocol
6     115140
17      1484
1        244
Name: count, dtype: int64

5. Top 5 source ports:
 sourceport
35218    3176
49198     744
43789     677
42932     397
57658     389
Name: count, dtype: int64

6. Average time between events: 0.74 seconds
```


And here are the two visualizations:

![](media/Figure_1.png "Figure 1")

![](media/Figure_2.png "Figure 2")

I think that is pretty impressive work from Claude. Even though I have very little pandas experience, I can ask Claude to give me some quick statistics based off of my firewall logs.

## Analysis

### 1. Events by source IP
```console
1. Events by source IP:
 source
172.233.32.48      32612
172.233.56.83      18280
172.233.56.7       17024
172.235.166.121     6276
165.154.0.174       4304
Name: count, dtype: int64
```

This first analysis lists the top 5 source IPs from the logs. The highest count is from the IP `172.233.32.48` with 32,612 hits! The second highest has half of that with 18,280!

Let's take a closer look at those IP addresses:

`172.233.32.48` appears to be from the US, owned by an "Akami Technologies, Inc." and "Linode." Searching this online brings me to [Linode's webpage](https://www.linode.com/), which is a cloud computing platform. Searching this IP on [threatstop](https://www.threatstop.com/check-ioc) shows that the IP is not a threat.

Searching up the next 3 IPs show the same info as the first IP.

The last IP `165.154.0.174` appears to be from Hong Kong, and the WHOIS record shows this IP owned by "UCLOUD INFORMATION TECHNOLOGY (HK) LIMITED." Threatstop says this IP is also not a threat.

It looks like these top 5 IPs are not malicious. What their purpose is, I do not know.

### 2. Events by Target Port

```console
2. Events by target port:
 targetport
23      77945
2222     1410
443       910
80        797
8728      552
Name: count, dtype: int64
```

Port 23 is overwhelmingly the most popular target port. Port 23 is the default port for [Telnet](https://en.wikipedia.org/wiki/Telnet), which is an outdated remote connection protocol.

Port 2222 [appears to be](https://tcp-udp-ports.com/port-2222.htm) an alternative port to port 22, which is SSH.

Port 443/80 is for HTTPS/HTTP, which is Hypertext Transfer Protocol (Secure). I don't believe any website is hosted there, so there was likely a 404 error.

Port 8728 [appears to be](https://www.speedguide.net/port.php?port=8728) the port to access the Operating System for a [MikroTik router](https://mikrotik.com/). This port must be unofficial, beacuse I do not see an official assignment to this port.

### 3. Events Per Hour

```console
3. Events per hour:
 hour
2024-09-20 00:00:00     6044
2024-09-20 01:00:00     1914
2024-09-20 02:00:00     1712
2024-09-20 03:00:00     1584
2024-09-20 04:00:00     1514
2024-09-20 05:00:00     1488
2024-09-20 06:00:00     1658
2024-09-20 07:00:00     1558
2024-09-20 08:00:00     1682
2024-09-20 09:00:00     1632
2024-09-20 10:00:00     1582
2024-09-20 11:00:00     1074
2024-09-20 12:00:00     1196
2024-09-20 13:00:00    77642
2024-09-20 14:00:00     1316
2024-09-20 15:00:00     1086
2024-09-20 16:00:00     1162
2024-09-20 17:00:00     1364
2024-09-20 18:00:00     1436
2024-09-20 19:00:00     1514
2024-09-20 20:00:00     1494
2024-09-20 21:00:00     2246
2024-09-20 22:00:00     1602
2024-09-20 23:00:00     1368
Name: count, dtype: int64
```

One P.M. was the hour with the most events, followed by 12 A.M. and 9 P.M. I cannot think of any significance of these three times of day. Midnight makes sense, since a server owner may be asleep, and maybe the other two hours are for different time zones.

### 4. Protocol Distribution

```console
4. Protocol distribution:
 protocol
6     115140
17      1484
1        244
Name: count, dtype: int64
```

According to [Wikipedia](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers), the protocol number identifies the protocol of the encapsulated packet, and it is found in the *Protocol* field of an IPv4 header and the *Next Header* of the IPv6 header.

Protocol number 6 is TCP (Transmission Control Protocol), which is by far the most popular protocol out of all events. Protocol number 17 refers to UDP (User Datagram Protocol), while protocol 1 refers to ICMP (Internet Control Message Protocol), which handles the `ping` command, among others.

### 5. Top 5 Source Ports

```console
5. Top 5 source ports:
 sourceport
35218    3176
49198     744
43789     677
42932     397
57658     389
Name: count, dtype: int64
```

These source ports are all unassigned except for 42932, which is used for some video games. Large, unassigned ports are likely used to obfuscate any attack.

### 6. Average Time Between Events

The average time between each event is 0.74 seconds.

## Conclusion

I cannot think of a reason why my honeypot was attacked this much only on this day. My quick analysis didn't yield much answers, but I was able to produce some simple statistics thanks to Claude and pandas. I plan to do more honeypot analyses in the future; likely not every month, but I hope to prepare more for the next event analysis.
