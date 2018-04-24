VAGRANTFILE_API_VERSION = "2"
file_to_disk1 = './disk1.vdi'

$script_node = <<SCRIPT
yum install vim wget nss-pam-ldapd -y
systemctl set-default graphical.target
echo "127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4" > /etc/hosts
echo "::1         localhost localhost.localdomain localhost6 localhost6.localdomain6" >> /etc/hosts
echo "172.31.200.100 node.example.local node" >> /etc/hosts
#rm -rf /etc/yum.repos.d/*
SCRIPT

$script_server = <<SCRIPT
yum update -y
yum install epel-release -y
yum install vim -y

### LDAP ###
yum install ipa-server -y
yum install sssd -y

systemctl stop chronyd
systemctl disable chronyd
echo "127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4" > /etc/hosts
echo "::1         localhost localhost.localdomain localhost6 localhost6.localdomain6" >> /etc/hosts
echo "172.31.200.200 server.example.local server" >> /etc/hosts
echo "172.31.200.100 node1.example.local node1" >> /etc/hosts
ipa-server-install --hostname=server.example.local -n example.local -r EXAMPLE.LOCAL -p redhat123 -a redhat123 -U
echo redhat123 | kinit admin
echo redhat123 | ipa user-add ldapuser1 --first user1 --last user1 --password
echo redhat123 | ipa user-add ldapuser2 --first user2 --last user2 --password
echo redhat123 | ipa user-add ldapuser3 --first user3 --last user3 --password
ipa host-add node1.example.local --force
ipa service-add nfs/node1.example.local --force
ipa service-add cifs/node1.example.local --force
ipa-getkeytab -s server.example.local -p nfs/node1.example.local -k /var/www/html/node1.keytab
ipa-getkeytab -s server.example.local -p cifs/node1.example.local -k /var/www/html/node1.keytab
ipa-getkeytab -s server.example.local -p host/node1.example.local -k /var/www/html/node1.keytab
mkdir -p /var/www/html/
chmod +r /var/www/html/*.keytab
chown apache:apache /var/www/html/*.keytab

#### NFS ###
yum install nfs-server
systemctl start nfs-server
systemctl enable nfs-server
systemctl enable nfs-secure
systemctl start nfs-secure
mkdir /home/ldapuser1 /home/ldapuser2 /home/ldapuser3 /secure
echo "/home/ldapuser1 172.31.200.0/24(rw)" > /etc/exports
echo "/home/ldapuser2 172.31.200.0/24(rw)" >> /etc/exports
echo "/home/ldapuser3 172.31.200.0/24(rw)" >> /etc/exports
echo "/secure 172.31.200.0/24(sec=krb5p,rw)" >> /etc/exports
exportfs -r

### SMB ###
yum install samba -y
mkdir /smbpath
chmod 777 /smbpath/
echo "[smbpath]" >> /etc/samba/smb.conf
echo "        comment = Samba share" >> /etc/samba/smb.conf
echo "        path = /smbpath" >> /etc/samba/smb.conf
echo "        writable = yes" >> /etc/samba/smb.conf
echo "        browseable = yes" >> /etc/samba/smb.conf
echo "        write list = fred" >> /etc/samba/smb.conf
echo "        valid users - fred" >> /etc/samba/smb.conf

useradd -s /sbin/nologin fred
(echo "redhat123"; echo "redhat123") | smbpasswd -a -s fred

systemctl start smb nmb
systemctl enable smb nmb

### WWW ###
yum install httpd -y
systemctl start httpd
systemctl enable httpd

#systemctl disable sshd.service
SCRIPT

Vagrant.configure(2) do |config|

  config.vm.define "node" do |conf|
    conf.vm.box = "centos/7"
    conf.vm.hostname = 'node.example.local'
    conf.vm.network "private_network", ip: "172.31.200.100"
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

