A quickstart for deployment a Single Node [Kubernetes](https://kubernetes.io/) v1.28.2 in Ubuntu 23.10 on [VMware Workstation](https://www.vmware.com/products/workstation-pro.html) via [Ansible Playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html).

## Playbook :notebook:

- Run & Enjoy :stuck_out_tongue:

  ```bash
  $ sudo ansible-playbook main.yml
  ```

- List of "Subject-to-Change" :spiral_notepad:

  ```bash
  # ./ansible.cfg
  remote_user = login_user_of_vm
  
  # ./inventory
  [group_name]
  1.2.3.4
  
  # ./bookstore/roles/docker/vars/main.yml
  sandbox_image: registry.aliyuncs.com/google_containers/pause:3.9
  
  # ./bookstore/roles/kubeadm/vars/main.yml
  image_repo: registry.aliyuncs.com/google_containers
  ver: v1.28.2
  cidr: 192.168.0.0/16
  ```

## VM​ :desktop_computer:

- [Image](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/23.10/ubuntu-23.10.1-desktop-amd64.iso)

- **2** CPU | **4GB** Mem | **50GB** Disk

- NAT VMnet8: `192.168.65.0/24` where Gateway: `192.168.65.2`

- After wizard

  ```bash
  ip=192.168.65.180
  mask=24
  gateway=192.168.65.2
  dns=192.168.65.2
  
  # Modify NIC & Up
  $ sudo nmcli con modify netplan-ens33 \
  ipv4.addresses "$ip/$mask" \
  ipv4.gateway "$gateway" \
  ipv4.dns "$dns" \
  ipv4.method manual \
  connection.autoconnect yes
  
  $sudo nmcli con up netplan-ens33
  
  # Install ssh
  $ sudo apt-get install ssh
  
  # Disable sudo pwd
  # sudo vim /etc/sudoers
  %sudo ALL=(ALL:ALL) NOPASSWD:ALL
  ```

## [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)​ :desktop_computer:

- wsl.conf

  ```bash
  # sudo vim /etc/wsl.conf 
  [network]
  generateResolvConf = true
  
  [boot]
  systemd=true
  
  # restart in Powershell
  wsl -l -v
  wsl -t <distro>
  wsl -d <distro>
  ```

- [Package Repository](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

  ```bash
  # backup
  $ sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
  
  # sudo vim /etc/apt/sources.list
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
  deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
  
  $ sudo apt-get update
  ```

- [ntp](https://www.ntppool.org/zone/cn)

  ```bash
  # Install
  $sudo apt-get install chrony
  
  # sudo vim /etc/chrony/chrony.conf
  server 0.cn.pool.ntp.org iburst
  server 1.cn.pool.ntp.org iburst
  server 2.cn.pool.ntp.org iburst
  server 3.cn.pool.ntp.org iburst
  
  # Verify
  chronyc sources
  ```

- zsh (Optional)

  ```bash
  # Install
  $ sudo apt-get install zsh
  $ sudo sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
  
  # chsh
  sudo chsh -s $(which zsh)
  
  # sudo vim ~/.zshrc
  plugins=(git zsh-autosuggestions zsh-syntax-highlighting copyfile tmux colored-man-pages extract)
  ```

- Proxy (Optional)

  ```bash
  # Append below
  # vim ~/.zshrc
  export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
  export proxyPort=7890
  alias proxy='
      export https_proxy="http://${hostip}:${proxyPort}";
      export http_proxy="http://${hostip}:${proxyPort}";
      export all_proxy="http://${hostip}:${proxyPort}";
      echo -e "Acquire::http::Proxy \"http://${hostip}:${proxyPort}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
      echo -e "Acquire::https::Proxy \"http://${hostip}:${proxyPort}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
  '
  alias unproxy='
      unset https_proxy;
      unset http_proxy;
      unset all_proxy;
      sudo sed -i -e "/Acquire::http::Proxy/d" /etc/apt/apt.conf.d/proxy.conf;
      sudo sed -i -e "/Acquire::https::Proxy/d" /etc/apt/apt.conf.d/proxy.conf;
  '
  
  # source 
  source ~/.zshrc
  
  # enable | disable proxy
  proxy
  unproxy
  ```

- ssh

  ```bash
  # RSA key-gen
  sshkeygen
  
  # Replace ${name}, ${ip} and ${user}; ${name} could be arbitaray
  # sudo vim ~/.ssh/config
  Host ${name}
    HostName ${ip}
    User ${user}
    
  # sudo vim ~/.zshrc
  alias ${name}="ssh ${name}"
  
  # source 
  source ~/.zshrc
  
  # Copy pub to VM
  ssh-copy-id ${name}
  
  # login
  ${name}
  ```

- [Ansible](https://docs.ansible.com/ansible/latest/os_guide/windows_faq.html#can-ansible-run-on-windows)

  ```bash
  apt-get update
  apt-get install python3-pip git libffi-dev libssl-dev -y
  pip install --user ansible pywinrm
  
  # Temp
  pip install -i https://pypi.tuna.tsinghua.edu.cn/simple ansible pywinrm
  
  # Verify
  ansible --version
  
  # dir for system role
  mkdir -p ~/.ansible/roles
  ```