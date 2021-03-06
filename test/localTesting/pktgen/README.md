# Packet generation to file

## What it is
Pktgen parses config in json format and generates packets according to it in pcap file that can be read by YANFF reader, Wireshark, tcpdump and other tools reading pcap files.

### Command-line options:
* --totalPackets sets the number of packets to generate, default value is 10000000
* --infile sets the name of the file with packet configurations, default value is "config.json"
* --outfile sets the name of the file to write output to, default value is "pkts_generated.pcap"

### Configuration syntax:
File should be a structure containing structure with ethernet configuration:
```json
{
    "ether": {
    }
}
```
Inside each header can be either data or next level header.
#### packet data configuration:
possible options are:
* "raw":
```json
"raw": {
  "data": "string with data that will be copied in packet.data"
}
```
* "randbytes":
```json
"randbytes": {
  "size": 50,
  "deviation": 10
}
```
size +-deviation random bytes will be generated, deviation field can be omitted
* "pdist":
```json
"pdist": [
    {
        "probability": 0.5,
        "randbytes":    {
            "size": 50,
            "deviation": 10
        }
    },
    {
        "probability": 0.5,
        "raw": {
            "data": "sfsfsfs"
        }
    }
]
```
pdist is an array of structures with data and probability of this data to be chosen
sum of probabilitis should be (0,1]

so, minimum config file example is:
```json
{
    "ether": {
        "raw": {
            "data": "111"
        }
    }
}
```

#### range structure:
```json
"range": {
    "min": "00:25:96:FF:FE:12",
    "start": "00:30:00:FF:FE:12",
    "max": "00:FF:96:FF:FE:12",
    "incr": 3
}
```
Range is available for IP, MAC addresses and TCP ports.
min and max fields are obligatory, start should be [min, max], by defauld start = min, incr by default is 0.
#### l2 config:
possible fields are:
* saddr which can be string or structure "range"
```json
"saddr": {
    "range": {
        "min": "00:25:96:FF:FE:12",
        "start": "00:30:00:FF:FE:12",
        "max": "00:FF:96:FF:FE:12",
        "incr": 3
    }
}
```
* daddr which can be string or structure "range"
```json
"daddr": "00:FF:96:FF:FE:12"
```
* VLAN tagging:
to each packet vlan tag can be added: "vlan-tci" and then numeric TCI information
```json
{
    "ether": {
                "saddr": "00:25:96:FF:FE:12",
                "daddr": "00:00:96:FF:00:00",
                "vlan-tci": 123,
                "ip": {
                    "version": 6,
                    "saddr": "2001:db8:a0b:12f0::",
                    "randbytes":    {
                        "size": 70
                    }
                }
            }
}
```

* l3 configuration or data, possible values are: "ip", "arp", "raw", "randbytes", "pdist":
```json
{
    "ether": {
                "saddr": {
                    "range": {
                        "min": "00:25:96:FF:FE:12",
                        "start": "00:30:00:FF:FE:12",
                        "max": "00:FF:96:FF:FE:12",
                        "incr": 3
                    }
                },
                "daddr": "00:FF:96:FF:FE:12",
                "pdist": [
                    {
                        "probability": 0.5,
                        "randbytes":    {
                            "size": 50,
                            "deviation": 10
                        }
                    },
                    {
                        "probability": 0.5,
                        "raw": {
                            "data": "sfsfsfs"
                        }
                    }
                ]
            }
}
```
#### l3 config:
Ip of 4 and 6 versions are supported
* "version" numeric vield with abailable values 4 or 6:
```json
"version": 6
```
* "saddr" sets a source ip address, can be string or range
for ip v4:
```json
"saddr": "1.1.127.1"
```
for ip v6:
```json
"saddr": "2001:db8:a0b:12f0::"
```
* "daddr" sets a destination ip address, can be string or range
for ip v4:
```json
"daddr": "1.1.127.1"
```
for ip v6:
```json
"daddr": "2001:db8:a0b:12f0::"
```
* l4 configuration or data, so probable values are: "tcp", "udp", "icmp", "raw", "randbytes" or "pdist"
```json
"ip": {
    "version": 4,
    "saddr": "1.1.127.1",
    "daddr": {
        "range": {
            "min": "1.1.1.1.",
            "max": "3.3.3.3"
        }
    },
    "raw": {
        "data": "023a0232ff9340x0340123"
    }
}
```

also arp packets are supported:
```json
{
    "ether": {
                "arp": {
                    "opcode": 1,
                    "gratuitous" : true,
                    "sha": "99:25:96:FF:FE:12",
                    "spa": "1.1.1.1"
                }
            }
}
```
* "opcode" is the operation code, supported only two values 1 for ARP Request and 2 for ARP Reply
* "gratuitous" is boolean field, can be ommited (false by default), but can be set to true to make announcement
* "sha" is a string with sender hardware address
* "tha" is a string with target hardware address
* "spa" is a string with sender protocol address
* "tpa" is a string with target protocol address
Ethernet source is set to sha by default, destination is broadcast.

#### l4 configuration:
##### "tcp" options:
* "sport" sets a source port can be numeric value of range
``` json
"sport": {
    "range": {
        "min": 1,
        "max": 8080,
        "incr": 100
    }
}
```
* "dport" sets a destination port can be numeric value of range
``` json
"dport": 1024
```
* "seq" sets a sequence number can be "incr"/"increasing" or "rand"/"random"
```json
 "seq": "increasing"
```
* "flags" sets tcp flags
```json
 "flags": ["ack", "psh"]
```
* data configuration: "raw", "randbytes", "pdist"
```json
"tcp": {
    "sport": {
        "range": {
            "min": 1,
            "max": 8080,
            "incr": 100
        }
    },
    "dport": 2000,
    "seq": "increasing",
    "flags": ["ack", "psh"],
    "pdist": [
        {
            "probability": 0.7,
            "randbytes":    {
                "size": 50,
                "deviation": 10
            }
        },
        {
            "probability": 0.2,
            "raw": {
                "data": "sfsfsfs"
            }
        }
    ]
}
```
##### udp options:
* "sport" sets a source port can be numeric value of range
``` json
"sport": {
    "range": {
        "min": 1,
        "max": 8080,
        "incr": 100
    }
}
```
* "dport" sets a destination port can be numeric value of range
``` json
"dport": 1024
```
* data configuration: "raw", "randbytes", "pdist"
```json
"udp": {
    "sport": 1,
    "dport": 2,
    "randbytes":    {
        "size": 100,
        "deviation": 20
    }
}
```
##### icmp options:
* "type" is a numeric field, sets type value in header:
```json
"type": 3
```
* "code" is a numeric field, sets code value in header:
```json
"code": 0
```
* "identifier" or "id" is a numeric field, sets identifier value in header:
```json
"id": 0
```
* "seq" or "seqNum" can be "incr"/"increasing" or "rand"/"random" sets a sequence number in header:
```json
"seq": "rand"
```
* data configuration: "raw", "randbytes", "pdist":
```json
"icmp": {
    "type": 10,
    "code": 1,
    "seq": "increasing",
    "pdist": [
        {
            "probability": 0.3,
            "randbytes":    {
                "size": 50,
                "deviation": 10
            }
        },
        {
            "probability": 0.3,
            "raw": {
                "data": "0000000000000000000000000000000000"
            }
        }
    ]
}
```

