# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  # Disale default folder sync
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Windows specific settings
  # Max time to wait for guest shutdown
  config.windows.halt_timeout = 25
  config.winrm.username = "vagrant"
  config.winrm.password = "vagrant"
  config.winrm.port = 25985

  # Ansible Control Node VM
  config.vm.define "ansiblectl", primary: true do |ansiblectl|
    ansiblectl.vm.box = "bento/ubuntu-20.04"
    ansiblectl.vm.host_name = "ansiblectl"
    ansiblectl.vm.provision "file", source: "ansible_files/", destination: "/tmp/ansible"
    ansiblectl.vm.provision "shell", inline: "mv /tmp/ansible /home/vagrant/ansible-wintest"
    ansiblectl.vm.provider "hyperv" do |hv|
        hv.memory = "1024"
        hv.vmname = "ansiblectl"
    end
    ansiblectl.vm.provision "shell", inline: "apt update"
    # Install Ansible
    ansiblectl.vm.provision "shell", inline: "apt install -y ansible"
    # Install Avahi mDNS for easy name resolution
    ansiblectl.vm.provision "shell", inline: "apt install -y avahi-daemon"
  end

  # Windows VM
  config.vm.define "wintest" do |wintest|
    wintest.vm.box = "StefanScherer/windows_2019"
    wintest.vm.guest = :windows
    wintest.vm.host_name = "wintest"
    wintest.vm.provider "hyperv" do |hv|
        hv.memory = "1024"
        hv.vmname = "wintest"
    end   

    # Install Chocolatey & Bounjour mDNS
    wintest.vm.provision "shell", privileged: "true", inline: "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))"
    # Install Bonjour mDNS for easy name resolution
    wintest.vm.provision "shell", privileged: "true", inline: "choco install bonjour -y" 
  end   

  # AWX VM
  config.vm.define "awx" do |awx|
    awx.vm.box = "bento/ubuntu-20.04"
    awx.vm.host_name = "awx"
    config.vm.provider :hyperv do |hv|
      hv.memory = 4096
      hv.cpus = 2
      hv.vmname = "awx"
    end
    awx.vm.provision "shell", inline: "apt update"
    # Install Avahi mDNS for easy name resolution
    awx.vm.provision "shell", inline: "apt install -y avahi-daemon"
    # Install Ansible
    awx.vm.provision "shell", inline: "apt install -y ansible"
    # Install & configure Docker CE, Compose, and related dependencies
    awx.vm.provision "shell", inline: "apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common"
    awx.vm.provision "shell", inline: "curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -"
    awx.vm.provision "shell", inline: "sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable\""
    awx.vm.provision "shell", inline: "echo \"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable\" | sudo tee /etc/apt/sources.list.d/docker.list"
    awx.vm.provision "shell", inline: "apt update"
    awx.vm.provision "shell", inline: "apt install -y docker-ce docker-ce-cli containerd.io"
    awx.vm.provision "shell", inline: "sudo usermod -aG docker $USER"
    awx.vm.provision "shell", inline: "newgrp docker"
    awx.vm.provision "shell", inline: "apt install -y docker-compose"
    # Install Node and NPM
    awx.vm.provision "shell", inline: "apt install -y nodejs npm"
    awx.vm.provision "shell", inline: "npm install npm --global"
    # Install AWX and related depedencies
    awx.vm.provision "shell", inline: "apt install -y python3-pip git pwgen vim"
    awx.vm.provision "shell", inline: "pip3 install requests==2.14.2"
    awx.vm.provision "shell", inline: "pip3 install docker-compose==1.25.0"
    awx.vm.provision "shell", inline: "git clone --depth 50 https://github.com/ansible/awx.git"
    awx.vm.provision "shell", inline: "ansible-playbook -i awx/installer/inventory awx/installer/install.yml"
    # Re-run the install play as workaround for https://github.com/ansible/awx/issues/6792
    awx.vm.provision "shell", inline: "ansible-playbook -i awx/installer/inventory awx/installer/install.yml"
  end
end  