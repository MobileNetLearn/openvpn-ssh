# openvpn-ssh

This works on raspberrypi.

The basic idea is run openvpn client and sshd service in docker. And run ssh -D to open one socks client. Then you can set proxy in your chrome with switchomega plugin.


# install
1. build docker images
```
# openvpn
cd openvpn
sudo docker build -f Dockerfile.arm -t armhf/openvpn:v1 .
# ssh
cd ssh
sudo docker build -f Dockerfile.arm -t armhf/ssh:v1 .
```
2. run openvpn
```
# -p 192.168.0.140:2222:22 :   expose 22 port to host 2222 port, 192.168.0.140 is your host IP
# -v /path/openvpn/config:/vpn : map host directory to container. It should contain the openvpn client config file and a file named user_passwd.txt which contain the key info.
# --up and --down  : better to set this.
# --route : route your host IP directly which means not throught openvpn. This is important.
sudo docker run -d -p 192.168.0.140:2222:22 --name openvpn --cap-add=NET_ADMIN --device /dev/net/tun -v /your/config/path/config:/vpn armhf/openvpn:v1 --config /vpn/config.ovpn --auth-user-pass /vpn/user_passwd.txt --auth-nocache --up /etc/openvpn/up.sh --down /etc/openvpn/down.sh --script-security 2 --route 192.168.0.140 255.255.255.255 172.17.0.1

```

3. run sshd
```
# first gen keys.
ssh-keygen
cat id_rsa.pub > authorized_keys
sudo chmod 600 authorized_keys
sudo chown root.root authorized_keys

# run sshd
# --net : means use openvpn container's network configure.
sudo docker run -d --name sshd --net=container:openvpn -v /path/to/authorized_keys:/root/.ssh/authorized_keys armhf/ssh:v1
```

4. run ssh
```
/usr/bin/ssh -i /path/to/id_rsa -N -D 192.168.0.140:1080 -p2222 root@192.168.0.140
```

