Vagrant.configure("2") do |config|
  # Глобальные настройки провайдера VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus   = 2
    vb.gui    = false   # Поменял на false (быстрее работает)
  end

  config.vm.boot_timeout = 900

  # ============================================================
  # Ubuntu 20.04 LTS (UFW)
  # ============================================================
  config.vm.define "ubuntu", primary: true do |m|
    m.vm.box = "ubuntu/jammy64"
    m.vm.hostname = "staging-ubuntu"
    m.vm.network  "private_network", ip: "192.168.56.10"
    
    m.vm.provider "virtualbox" do |vb|
      vb.name = "firewall-staging-ubuntu"
    end

    # Shell provisioning для Ubuntu
    m.vm.provision "shell", inline: <<-SHELL
      echo "nameserver 8.8.8.8" > /etc/resolv.conf
      echo "==> Настройка Ubuntu..."
      apt-get update -y
      apt-get install -y python3 python3-pip sshpass
      pip3 install --upgrade pip
      pip3 install ansible
      echo "==> Подготовка Ubuntu завершена!"
    SHELL
  end

  # ============================================================
  # CentOS 8 (Firewalld)
  # ============================================================
  config.vm.define "centos" do |m|
    m.vm.box = "centos/8"
    m.vm.hostname = "staging-centos"
    m.vm.network  "private_network", ip: "192.168.56.20"
    
    m.vm.provider "virtualbox" do |vb|
      vb.name = "firewall-staging-centos"
    end

    # Shell provisioning для CentOS
    m.vm.provision "shell", inline: <<-SHELL
      echo "nameserver 8.8.8.8" > /etc/resolv.conf
      echo "==> Настройка CentOS..."
      cd /etc/yum.repos.d/
      sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
      sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
      
      yum clean all
      yum install -y python3 python3-pip
      pip3 install --upgrade pip
      pip3 install ansible
      
      NM_CONN=$(nmcli -t -f NAME connection show | grep -v "System eth0" | head -n 1)
      if [ ! -z "$NM_CONN" ]; then
        nmcli connection up "$NM_CONN"
      fi
      echo "==> Подготовка CentOS завершена!"
    SHELL

    # ============================================================
    # Запуск Ansible ПЛ ЕЙБУКА (ТОЛЬКО ОДИН РАЗ!)
    # ============================================================
    m.vm.provision "ansible" do |ansible|
      ansible.playbook        = "playbook.yml"
      ansible.inventory_path  = "inventory/staging.yml"
      ansible.limit           = "all"
      ansible.verbose         = "v"
      ansible.extra_vars      = {
        debug_mode: true,
        firewall_backup: true
      }
    end
  end
end