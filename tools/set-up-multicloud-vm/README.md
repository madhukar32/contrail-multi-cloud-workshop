### Setting up contrail-multi-cloud gateway VM

##### Get contrail-multi-cloud source code details


1. Git clone contrail-multi-cloud

```bash
git clone https://github.com/Juniper/contrail-multi-cloud.git
```

2. Go to contrail-multi-cloud directory

```bash
cd contrail-multi-cloud
```

4. Enable forwarding from docker containers to outside world

```bash
sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
```

5. Login to juniper's docker registry

```bash
docker login -u <username> -p <password> hub.juniper.net/contrail
```

##### [Links for workshop deployment to attendees](../../workshop/apac/README.md)
