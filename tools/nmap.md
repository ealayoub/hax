# nmap
nmap is a network scanner used to discover hosts and services on a network
## basic scan
```bash
nmap {target_ip}
```
## useful flags
- -p-: scan all ports
- -sV: service and version detection
- -sC: default scripts
- -sS: SYN scan (stealthy)
- -T{0 to 5}: speed of scan. Higher = more risky

