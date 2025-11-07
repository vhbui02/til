# AWS EC2 Provisioning Cheatsheet

<!-- tl;dr starts -->

The reason that I'm using AWS is because Cloudflare doesn't have compute engine service and its free tier counterpart is quite generous. I've decided to properly learn it.

<!-- tl;dr ends -->

## `t2.nano` vs `t3.nano`

| Resource               | t2.nano                | t3.nano                |
| ---------------------- | ---------------------- | ---------------------- |
| Region                 | All regions            | Same                   |
| Availability           | 750 hours/month        | Same                   |
| vCPU                   | 1                      | 2                      |
| RAM (GiB)              | 0.5                    | 1                      |
| EBS Storage            | 30 GB                  | 30 GB                  |
| CPU credits/hour       | 6                      | Same                   |
| Maximum CPU credits    | 144 (24 hours non-use) | 288 (48 hours non-use) |
| Egress                 | 100GB/month            | Same                   |
| Unlimited mode default | No                     | Yes (**TURN IT OFF**)  |
| # of Public IPv4       | 1                      | Same                   |
| # of Public IPv6       | 1                      | Same                   |
| Duration               | 12 months              | 12 months              |

## Provisioning Script

At the local machine:

```sh
export EC2_USER=ubuntu # or ec2-user, ...
export EC2_DOMAIN=ec2-54-46-1-22.ap-east-1.compute.amazonaws.com # or public static IPv4 address

# ssh'ing!
ssh "$EC2_USER@$EC2_DOMAIN"
```

At the remote machine:

```sh
# ===================================================================== #
# Misc                                                                  #
# ===================================================================== #

# set timezone to GMT+7
sudo timedatectl set-timezone "Asia/Bangkok"

# 32*128M = 4096MB = 4GB
sudo dd if=/dev/zero of=/swapfile bs=128M count=32
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon -s
echo "/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab

# ===================================================================== #
# Default User Privileged Management                                    #
# ===================================================================== #

# Add password for ubuntu
# https://serverfault.com/q/728831/1138548
sudo passwd "$USER"

# CAUTION: DO NOT EXECUTE THE `sed` COMMAND IF YOU HAVEN"T SET PASSWORD FOR `ubuntu` YET
sudo sed --in-place "/^$USER /s/NOPASSWD://g" /etc/sudoers.d/90-cloud-init-users

# Persist `sudo` by an hour
echo "Defaults      env_reset,timestamp_timeout=3600" | sudo tee -a /etc/sudoers.d/00-persist-sudo

# ===================================================================== #
# Fish Shell                                                            #
# ===================================================================== #

# Install Fish shell and set Fish as the default shell
sudo apt-add-repository ppa:fish-shell/release-4
sudo apt update
sudo apt install fish -y
sudo chsh -s /usr/bin/fish $USER

# Add custom binary directory into PATH variable
mkdir -pv ~/.local/bin
fish_add_path -v ~/.local/bin

# Install Fish package manager
fish -c 'curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher'

# Install Fish extensions and their dependencies
sudo apt install bat fd-find fzf
ln -s $(which fdfind) ~/.local/bin/fd
fisher install jorgebucaran/autopair.fish jethrokuan/z edc/bass patrickf1/fzf.fish

# change the prompt structure
fish_config prompt list # in case of changing your mind
fish_config prompt choose informative
fish_config prompt save

# change the theme color
fish_config theme list # in case of changing your mind
fish_config theme choose "Tomorrow Night Bright"
fish_config theme save

# remove the greeting
echo 'function fish_greeting -d "Remove the greetings"
end' | tee ~/.config/fish/functions/fish_greeting.fish

# add a new line after each prompt execution
echo 'if status is-interactive
  function postexec_fish --on-event fish_postexec -d "Add a newline after each command"
    echo
  end
end' | tee ~/.config/fish/config.fish

# ===================================================================== #
# Docker                                                                #
# ===================================================================== #

# https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world

# start these services on boot
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

# avoid writing `sudo docker ...`
sudo usermod -aG docker $USER

# config log rotation to avoid accumulating disk space
echo '{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}' | sudo tee /etc/docker/config.json

# Restart Docker to apply changes
sudo systemctl restart docker

# ===================================================================== #
# Tailscale                                                             #
# ===================================================================== #

# https://tailscale.com/kb/1476/install-ubuntu-2404
export CODENAME="noble"

curl -fsSL "https://pkgs.tailscale.com/stable/ubuntu/$CODENAME.noarmor.gpg" | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL "https://pkgs.tailscale.com/stable/ubuntu/$CODENAME.tailscale-keyring.list" | sudo tee /etc/apt/sources.list.d/tailscale.list

sudo apt update
sudo apt install tailscale -y
sudo tailscale set --operator="${USER}" --ssh

exit
```

Then at the local machine:

```sh
# add personal nano configuration
tailscale ssh "$EC2_USER@$EC2_DOMAIN" 'cat > ~/.nanorc' < ~/.nanorc
tailscale ssh "root@$EC2_DOMAIN" 'cat > /root/.nanorc' < ~/.nanorc

# keep connections alive longer, avoid freezing
tailscale ssh "root@$EC2_DOMAIN" '
  sed -i \
      -e "s/^#ClientAliveInterval.*/ClientAliveInterval 20/" \
      -e "s/^#ClientAliveCountMax.*/ClientAliveCountMax 180/" \
      /etc/ssh/sshd_config
  systemctl restart sshd
'
```

## Install FFmpeg inside AWS EC2 using Amazon Linux 2023

```sh
# SSH into root user
docker exec -it tailscale tailscale ssh root@remotelab

# Install required dependencies
sudo yum install yasm nasm \
autoconf automake bzip2 bzip2-devel cmake freetype-devel \
gcc gcc-c++ git libtool make pkgconfig zlib-devel

# Download and extract FFmpeg source
curl -O -L https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 -C /tmp
tar xjvf /tmp/ffmpeg-snapshot.tar.bz2 -C /tmp
cd /tmp/ffmpeg

# Configure FFmpeg build
# Docs: https://gist.github.com/omegdadi/6904512c0a948225c81114b1c5acb875
./configure --prefix="$HOME/ffmpeg" \
  --bindir="$HOME/ffmpeg/bin" \
  --extra-cflags="-I$HOME/ffmpeg/include -fstack-protector-strong -fpie -pie -Wl,-z,relro,-z,now -D_FORTIFY_SOURCE=2" \
  --extra-ldflags="-L$HOME/ffmpeg/lib" \
  --extra-libs=-lpthread \
  --extra-libs=-lm \
  --enable-libfreetype \
  --disable-static \
  --enable-shared \
  --enable-rpath
  # --enable-gpl \
  # --enable-version3 \
  # --enable-nonfree
```

## IP address

By default, an AWS EC2 instance can have both static and dynamic IPv4 addresses:

### Dynamic IPv4

- Public: when you stop/start the instance, a new public IPv4 is newly assigned.
- Private: always assigned from the VPC subnet range and remains static within the VPC.

### Static IPv4

- Elastic IP (or EIP): set up a static public IPv4. Note that it's only free when it's associated with a running EC2 instance. If your EC2 is dead, make sure your Elastic IP shares the same fate.
- Private, static IP: You can assign a specific static private IP when launching the instance (?)
