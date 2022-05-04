# Local Environment Setup
Installation and configuration instructions for the local work environment, primarily Windows Subsystem for Linux

## WSL2 setup
Setup for WSL2, in order to use Docker. However, web traffic on VPN, in WSL2,
will most likely get dropped (which effects package installs, Git push/pulls,
and data sourcing). I would advise not going down the rabbit hole to
figure out a solution. Consider installing an additional WSL image,
set as version 1, for VPN web requests.
(Consider creating a local web proxy? See Python requests and socket libraries.)

### Install WSL2 Debian
See https://docs.microsoft.com/en-us/windows/wsl/install.

### Check DNS resolution
First, change permissions on ping to user read, write, and execute.
```
which ping
sudo chmod 4711 /bin/ping
```

```
ping google.com
```

Temporary failure in name resolution? Configure DNS resolver.
(Although it still seems to get reset.)
```
echo -e "[network]\ngenerateResolvConf = false" | sudo tee -a /etc/wsl.conf
```

Some documentation added: ```sudo unlink /etc/resolv.conf```.

Edit the DNS server to CloudFlare or Google, etc.
```
echo nameserver 1.1.1.1 | sudo tee /etc/resolv.conf
```

### Windows WSL command reference
In CMD:
```
wsl --shutdown
wsl --list --verbose
wsl --set-default-version 2
wsl --set-version {distro_name} 1
```

References:
- https://gist.github.com/coltenkrauter/608cfe02319ce60facd76373249b8ca6?permalink_comment_id=4045147
- https://dev.to/bowmanjd/install-docker-on-windows-wsl-without-docker-desktop-34m9

### Update
```
sudo apt update
sudo apt upgrade -y
```

### Install some Linux packages
Keep this list to a minimum, keeping OS as clean as possible in order to
avoid package dependency and breakage issues. Rely on Docker containers.
```
sudo apt install htop -y
sudo apt install vim -y
sudo apt install zip unzip -y
sudo apt install python3 python3-pip -y                     # Perhaps not needed
```
Some other isntalls are in the Docker install section.

### Install Docker
Dependencies.
```
sudo apt install --no-install-recommends \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg2
```

Switch to legacy IP tables (for Debian at least).
```
sudo apt install --reinstall iptables -y
sudo update-alternatives --config iptables
```
Select iptables-legacy.

Repository configuration.
```
curl -fsSL https://download.docker.com/linux/debian/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
. /etc/os-release
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker.
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

Add user to docker group.
```
sudo usermod -aG docker $USER
```

Run the daemon.
```
sudo dockerd
```

Test running test image. (Ensure resolv.conf is configured.)
```
docker run hello-world
```

References:
- https://dev.to/bowmanjd/install-docker-on-windows-wsl-without-docker-desktop-34m9
- https://docs.docker.com/engine/install/debian/

### Install Docker Compose
Download binary from Docker Github and put into Docker plugins directory
Get the DL URL copying from https://github.com/docker/compose/releases
```
DL_URL=https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-x86_64
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL $DL_URL -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

Make executable permissions and test.
```
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
docker compose version
```

Reference:
https://docs.docker.com/compose/install/

### Set WSL2 symbolic links to Windows resources
```
ln -s /mnt/c/Users/myusername/path/to/directory /full/path/to/new_link_name
```
