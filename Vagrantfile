# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
BOX_NAME = "CentOS-6.5-x86_64-minimal.box"
BOX_PATH = "./packer/builds/#{BOX_NAME}"

if !FileTest.exists?(BOX_PATH)
  system("cd packer && packer build template.json")
end

$add_user = <<EOF

  grep "anoonna" /etc/passwd;

  if [ "$?" -eq 1 ]; then
    adduser -m anoonna
  fi

  grep "anoonna" /etc/sudoers;

  if [ "$?" -eq 1 ]; then
    echo "anoonna    ALL=(ALL)       NOPASSWD:ALL" >> /etc/sudoers
  fi

EOF

$setup_ssh_key = <<EOF

  if [ ! -e "/home/anoonna/.ssh" ]; then
    mkdir -p /home/anoonna/.ssh
    chmod 0700 /home/anoonna/.ssh
    curl -o /home/anoonna/.ssh/authorized_keys https://github.com/futoase.keys
    chown -R anoonna:anoonna /home/anoonna/.ssh
    chmod 0600 /home/anoonna/.ssh/authorized_keys
  fi

EOF

$mk_deploy_dir = <<EOF

  if [ ! -e "/var/app" ]; then
    mkdir -p /var/app
    chown anoonna:anoonna /var/app
  fi

EOF

$install_nc = <<EOF
  yum install -y nc;
EOF

$stop_the_service = <<EOF
  service iptables stop
EOF

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "CentOS-6.5-x86_64-minimal.box"
  config.vm.box_url = "packer/builds/CentOS-6.5-x86_64-minimal.box"

  config.vm.network :private_network, ip: "192.168.33.50"
  config.ssh.forward_agent = true

  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provider :virtualbox do |vb|
    vb.gui = false
    vb.customize ["modifyvm", :id, "--memory", "384"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  config.vm.provision "shell", inline: $add_user
  config.vm.provision "shell", inline: $setup_ssh_key
  config.vm.provision "shell", inline: $mk_deploy_dir
  config.vm.provision "shell", inline: $install_nc
  config.vm.provision "shell", inline: $stop_the_service

end
