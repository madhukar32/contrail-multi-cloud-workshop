### Installing docker

##### INFO

```console
docker version: 18.03.1
```

##### Steps to install docker

1. Create docker centos repo

```bash
sudo bash -c 'cat <<EOF > /etc/yum.repos.d/dockerrepo.repo
[dockerrepo]
baseurl = https://yum.dockerproject.org/repo/main/centos/7
gpgcheck = 1
gpgkey = https://yum.dockerproject.org/gpg
name = Docker Repository
EOF'
```

2. Install docker selinux

```bash
sudo yum install -y docker-engine-selinux-17.03.1.ce
```

3. Install docker

```bash
sudo yum install -y docker-engine-17.03.1.ce
```

3. Start docker service

```bash
sudo systemctl start docker
```
