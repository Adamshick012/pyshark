#pyshark

Python wrapper for tshark, allowing python packet parsing using wireshark dissectors.

Extended documentation: http://kiminewt.github.io/pyshark

There are quite a few python packet parsing modules, this one is different because it doesn't actually parse any packets, it simply uses tshark's (wireshark command-line utility) ability to export XMLs to use its parsing.

This package allows parsing from a capture file or a live capture, using all wireshark dissectors you have installed.
Tested on windows/linux.

##Usage

###Reading from a capture file:

```python
>>> import pyshark
>>> cap = pyshark.FileCapture('/tmp/mycapture.cap')
>>> cap
<FileCapture /tmp/mycapture.cap (589 packets)>
print cap[0]
Packet (Length: 698)
Layer ETH:
        Destination: BLANKED
        Source: BLANKED
        Type: IP (0x0800)
Layer IP:
        Version: 4
        Header Length: 20 bytes
        Differentiated Services Field: 0x00 (DSCP 0x00: Default; ECN: 0x00: Not-ECT (Not ECN-Capable Transport))
        Total Length: 684
        Identification: 0x254f (9551)
        Flags: 0x00
        Fragment offset: 0
        Time to live: 1
        Protocol: UDP (17)
        Header checksum: 0xe148 [correct]
        Source: BLANKED
        Destination: BLANKED
  ...
```

#### Other options

* **lazy**: Whether to lazily get packets from the cap file or read all of them 
immediately.
* **param keep_packets**: Whether to keep packets after reading them via next(). 
Used to conserve memory when reading large caps (can only be used along with 
the "lazy" option!)
* **param input_file**: Either a path or a file-like object containing either a 
packet capture file (PCAP, PCAP-NG..) or a TShark xml.
* **param bpf_filter**: A BPF (tcpdump) filter to apply on the cap before reading.
* **param display_filter**: A display (wireshark) filter to apply on the cap 
before reading it.
* **param only_summaries**: Only produce packet summaries, much faster but includes 
very little information
* **param decryption_key**: Key used to encrypt and decrypt captured traffic.
* **param encryption_type**: Standard of encryption used in captured traffic (must 
be either 'WEP', 'WPA-PWD', or 'WPA-PWK'. Defaults to WPA-PWK.
  
###Reading from a live interface:

```python
>>> capture = pyshark.LiveCapture(interface='eth0')
>>> capture.sniff(timeout=50)
>>> capture
<LiveCapture (5 packets)>
>>> capture[3]
<UDP/HTTP Packet>

for packet in capture.sniff_continuously(packet_count=5):
    print 'Just arrived:', packet
```

#### Other options

* **param interface**: Name of the interface to sniff on. If not given, takes 
the first available.
* **param bpf_filter**: BPF filter to use on packets.
* **param display_filter**: Display (wireshark) filter to use.
* **param only_summaries**: Only produce packet summaries, much faster but 
includes very little information
* **param decryption_key**: Key used to encrypt and decrypt captured traffic.
* **param encryption_type**: Standard of encryption used in captured traffic 
(must be either 'WEP', 'WPA-PWD', or 'WPA-PWK'. Defaults to WPA-PWK).

###Reading from a live remote interface:

```python
>>> capture = pyshark.RemoteCapture('192.168.1.101', 'eth0')
>>> capture.sniff(timeout=50)
>>> capture
```

#### Other options

* **param remote_host**: The remote host to capture on (IP or hostname). 
Should be running rpcapd.
* **param remote_interface**: The remote interface on the remote machine to 
capture on. Note that on windows it is not the device display name but the 
true interface name (i.e. \\Device\\NPF_..).
* **param remote_port**: The remote port the rpcapd service is listening on
* **param bpf_filter**: A BPF (tcpdump) filter to apply on the cap before 
reading.
* **param only_summaries**: Only produce packet summaries, much faster but 
includes very little information
* **param decryption_key**: Key used to encrypt and decrypt captured traffic.
* **param encryption_type**: Standard of encryption used in captured traffic 
(must be either 'WEP', 'WPA-PWD', or 'WPA-PWK'. Defaults to WPA-PWK).

###Accessing packet data:

Data can be accessed in multiple ways. 
Packets are divided into layers, first you have to reach the appropriate layer and then you can select your field.

All of the following work:

```python
>>> packet['ip'].dst
192.168.0.1
>>> packet.ip.src
192.168.0.100
>>> packet[2].src
192.168.0.100
```

###Decrypting packet captures

Pyshark supports automatic decryption of traces using the WEP, WPA-PWD, and WPA-PSK standards (WPA-PWD is the default). 

```python
>>> cap1 = pyshark.FileCapture('/tmp/capture1.cap', decryption_key='password')
>>> cap2 = pyshark.LiveCapture(interface='wi0', decryption_key='password', encryption_type='wpa-psk')
```

A tuple of supported encryption standards, SUPPORTED_ENCRYPTION_STANDARDS, 
exists in each capture class.

```python
>>> pyshark.FileCapture.SUPPORTED_ENCRYPTION_STANDARDS
('wep', 'wpa-pwd', 'wpa-psk')
>>> pyshark.LiveCapture.SUPPORTED_ENCRYPTION_STANDARDS
('wep', 'wpa-pwd', 'wpa-psk')
```
