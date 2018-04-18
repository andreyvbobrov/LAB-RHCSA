VAGRANTFILE_API_VERSION = "2"
file_to_disk1 = './disk1.vdi'

$script_node = <<SCRIPT
yum install vim nss-pam-ldapd -y
systemctl set-default graphical.target
#rm -rf /etc/yum.repos.d/*
SCRIPT

$script_server = <<SCRIPT
yum update -y
yum install epel-release -y
yum install vim -y
yum install ipa-server -y
yum install sssd -y

systemctl stop chronyd
systemctl disable chronyd
echo "127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4" > /etc/hosts
echo "::1         localhost localhost.localdomain localhost6 localhost6.localdomain6" >> /etc/hosts
echo "172.31.200.200 server.example.local server" >> /etc/hosts
ipa-server-install --hostname=server.example.local -n example.local -r EXAMPLE.LOCAL -p redhat123 -a redhat123 -U
echo redhat123 | kinit admin
echo redhat123 | ipa user-add ldapuser1 --first user1 --last user1 --password
echo redhat123 | ipa user-add ldapuser2 --first user2 --last user2 --password
echo redhat123 | ipa user-add ldapuser3 --first user3 --last user3 --password

#systemctl disable sshd.service
SCRIPT

Vagrant.configure(2) do |config|

  config.vm.define "node" do |conf|
    conf.vm.box = "centos/7"
    conf.vm.hostname = 'node.example.local'
    conf.vm.network "private_network", type: "dhcp"
    conf.vm.provision "shell", inline: $script_node
    config.vm.provider "virtualbox" do |v|  
    unless File.exist?(file_to_disk1)
      v.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
      v.customize ['createhd', '--filename', file_to_disk1, '--size', 10 * 1024]
      v.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk1]
    end
    end
  end

  config.vm.define "server" do |conf|
    conf.vm.box = "centos/7"
    conf.vm.hostname = 'server.example.local'
    conf.vm.network "private_network", ip: "172.31.200.200"
    conf.vm.provision "shell", inline: $script_server
  end

end

