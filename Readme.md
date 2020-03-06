# Proc
1. Install packages

```bash
yum install epel-release -y
yum install openvpn -y
yum install easy-rsa -y
```

```bash
easy-rsa init-pki
easy-rsa build-ca
easy-rsa build-server-full server nopass
easy-rsa build-client-full client
easy-rsa gen-dh
openvpn --genkey --secret ta.key
```

```bash
yum install fail2ban -y
systemctl start fail2ban
```

2. Set up config (only added/modified lines are shown here)

- file: /etc/openvpn/server/server.conf
  ```
  tls-server

  ca /root/pki/ca.crt
  cert /root/pki/issued/server.crt
  key /root/pki/private/server.key

  dh /root/pki/dh.pem

  tls-auth /root/pki/ta.key 0

  user nobody
  group nobody
  ```

<!-- - iptables script: -->
<!--   ```bash -->
<!--   #! /usr/bin/sh -->
<!--   # Flushing all rules -->
<!--   iptables -F -->
<!--   iptables -X -->
<!--   # Setting default filter policy -->
<!--   iptables -P INPUT DROP -->
<!--   iptables -P OUTPUT DROP -->
<!--   iptables -P FORWARD DROP -->
<!--   # Allow unlimited traffic on loopback -->
<!--   iptables -A INPUT -i lo -j ACCEPT -->
<!--   iptables -A OUTPUT -o lo -j ACCEPT -->
<!--   # Allow incoming ssh only -->
<!--   iptables -A INPUT -p tcp --dport 22 -j ACCEPT -->
<!--   iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT -->
<!--   ``` -->

3. Client conf
- file: client.ovpn (the keys are in the same location)
  ```
  tls-client
  remote 192.168.99.104
  ca ca.crt
  key client.key
  cert client.crt
  tls-auth ta.key 1
  remote-cert-tls server
  dev tun
  ```

4. Hardening
- passwords
  entropy: +80 bits
- ports whithelisting
  only 22 and 1194
- firewalld
- SeLinux
  enforcing
