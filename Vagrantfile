# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
sudo yum install wget --y
sudo yum install ntp --y
sudo yum install screen --y
sudo yum install epel-release --y
sudo yum install vim --y
sudo yum install iptables --y
sudo yum install iptables-utils --y
sudo yum install ncurses-term --y
sudo yum install iptables-services --y
sudo yum install consul --y
sudo yum install docker --y
sudo yum install curl --y
sudo yum install unzip --y
sudo usermod -aG docker vagrant
sudo docker --version
# Packages required for nomad & consul
echo "Installing Nomad..."
NOMAD_VERSION=0.10.4
cd /tmp/
curl -sSL https://releases.hashicorp.com/nomad/${NOMAD_VERSION}/nomad_${NOMAD_VERSION}_linux_amd64.zip -o nomad.zip
unzip nomad.zip
sudo install nomad /usr/bin/nomad
sudo mkdir -p /etc/nomad.d
sudo chmod a+w /etc/nomad.d
echo "Installing Consul..."
CONSUL_VERSION=1.6.4
curl -sSL https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip > consul.zip
unzip /tmp/consul.zip
sudo install consul /usr/bin/consul
(
cat <<-EOF
  [Unit]
  Description=consul agent
  Requires=network-online.target
  After=network-online.target
  [Service]
  Restart=on-failure
  ExecStart=/usr/bin/consul agent -dev
  ExecReload=/bin/kill -HUP $MAINPID
  [Install]
  WantedBy=multi-user.target
EOF
) | sudo tee /etc/systemd/system/consul.service
sudo systemctl enable consul.service
sudo systemctl start consul
for bin in cfssl cfssl-certinfo cfssljson
do
  echo "Installing $bin..."
  curl -sSL https://pkg.cfssl.org/R1.2/${bin}_linux-amd64 > /tmp/${bin}
  sudo install /tmp/${bin} /usr/local/bin/${bin}
done
/usr/bin/nomad -autocomplete-install
SCRIPT

NODES = 1
DISKS = 1
MEMORY = 4096 
CPUS = 1 
NESTED = true

### TYPE HERE A PREFIX ###
PREFIX = "grant-nomad"

Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vm.box = "centos/7"
  config.vm.hostname = "nomad"
  config.vm.provision "shell", inline: $script, privileged: false

  # Override
  config.vm.provider :libvirt do |v,override|
      override.vm.synced_folder '.', '/home/vagrant/sync', disabled: true
  end

  
  # Expose the nomad api and ui to the host
  config.vm.network "forwarded_port", guest: 4646, host: 4646, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.network :private_network, ip: "192.168.30.70"

  # Increase memory for libvirt
  config.vm.provider :libvirt do  |lv|
    driverletters = ('b'..'z').to_a
    lv.storage :file, :device => "vdb", :path => "#{PREFIX}-disk-1-1.disk", :size => '1024G'
    lv.memory = MEMORY
    lv.cpus = CPUS
    lv.nested = NESTED
  end

  config.vm.provision :ansible do |ansible|
    ansible.limit = "all"
    ansible.playbook = "site.yml"
    ansible.groups = {
        "master" => ["#{PREFIX}-master"],
    }
  end
end
