# Gobuster
This is a file/directory brute-forcing tool to discover hidden paths on web servers.

## Usage:
```bash
gobuster dir -u http://<target_ip> -w <wordlist>
```

## Common Options:
- -u: target url
- -w: wordlist
- -x: file extension to check (.php, .txt)
- -t: threads
- -o: save output to file

## vhost enumeration:
gobuster can also brute force virtual host if the target uses name-based virtual hosting.
### example:
```bash
gobuster vhost -u http://<target_ip> -w <wordlist>
```