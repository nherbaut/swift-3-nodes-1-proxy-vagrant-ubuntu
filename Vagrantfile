# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.


boxes = [
    {
        :name => "proxy1",
        :eth1 => "192.168.236.60",
        :eth2 => "192.168.252.60",
        :mem => "512",
        :cpu => "1",
        :nodetype => "proxy"
    },

	
    {
        :name => "object1",
        :eth1 => "192.168.236.70",
        :eth2 => "192.168.252.70",
        :mem => "512",
        :cpu => "1",
        :nodetype => "object"
    },
    {
        :name => "object2",
        :eth1 => "192.168.236.71",
        :eth2 => "192.168.252.71",
        :mem => "512",
        :cpu => "1",
        :nodetype => "object"
    },
    {
        :name => "object3",
        :eth1 => "192.168.236.72",
        :eth2 => "192.168.252.72",
        :mem => "512",
        :cpu => "1",
        :nodetype => "object"
    }

]



$commonscript = <<COMMONSCRIPT


apt-get install ubuntu-cloud-keyring
echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" "trusty-updates/kilo main" > /etc/apt/sources.list.d/cloudarchive-kilo.list
apt-get update && apt-get upgrade --yes


cat << EOF >> /etc/hosts
192.168.236.60 proxy1
192.168.236.70 object1
192.168.236.71 object2
192.168.236.72 object3
EOF

# Environment variables used in scripts
ETH1_IP=`ip route get 192.168.236.0/24 | awk 'NR==1 {print $NF}'`
echo export ETH1_IP=$ETH1_IP > /etc/profile.d/eth1-ip.sh
ETH2_IP=`ip route get 192.168.252.0/24 | awk 'NR==1 {print $NF}'`
echo export ETH2_IP=$ETH2_IP > /etc/profile.d/eth2-ip.sh
echo `whoami` >> whoami2

COMMONSCRIPT


$proxyscript = <<PROXYSCRIPT


#allow root access
sudo sed -i "s/PermitRootLogin without-password/PermitRootLogin yes/g" /etc/ssh/sshd_config
sudo echo "root:vagrant" | sudo chpasswd
service ssh restart




apt-get install swift swift-proxy python-swiftclient memcached --yes
sudo sed -i "s/127.0.0.1/0.0.0.0/g" /etc/memcached.conf
mkdir /etc/swift

echo "[DEFAULT]" > /etc/swift/proxy-server.conf
echo "bind_ip = 192.168.236.60" >> /etc/swift/proxy-server.conf
echo "bind_port = 8080" >> /etc/swift/proxy-server.conf
echo "workers = 8" >> /etc/swift/proxy-server.conf
echo "user = swift" >> /etc/swift/proxy-server.conf
echo "" >> /etc/swift/proxy-server.conf
echo "[pipeline:main]" >> /etc/swift/proxy-server.conf
echo "pipeline = healthcheck cache tempauth proxy-server" >> /etc/swift/proxy-server.conf
echo "" >> /etc/swift/proxy-server.conf
echo "[app:proxy-server]" >> /etc/swift/proxy-server.conf
echo "use = egg:swift#proxy" >> /etc/swift/proxy-server.conf
echo "allow_account_management = true" >> /etc/swift/proxy-server.conf
echo "account_autocreate = true" >> /etc/swift/proxy-server.conf
echo "" >> /etc/swift/proxy-server.conf
echo "[filter:cache]" >> /etc/swift/proxy-server.conf
echo "use = egg:swift#memcache" >> /etc/swift/proxy-server.conf
echo "memcache_servers = 192.168.252.60:11211" >> /etc/swift/proxy-server.conf
echo "" >> /etc/swift/proxy-server.conf
echo "[filter:catch_errors]" >> /etc/swift/proxy-server.conf
echo "use = egg:swift#catch_errors" >> /etc/swift/proxy-server.conf
echo "" >> /etc/swift/proxy-server.conf
echo "[filter:healthcheck]" >> /etc/swift/proxy-server.conf
echo "use = egg:swift#healthcheck" >> /etc/swift/proxy-server.conf
echo "" >> /etc/swift/proxy-server.conf
echo "[filter:tempauth]" >> /etc/swift/proxy-server.conf
echo "use = egg:swift#tempauth" >> /etc/swift/proxy-server.conf
echo "# user_<tenant>_<username> = <password> <privileges> " >> /etc/swift/proxy-server.conf
echo "user_admin_admin = admin .admin .reseller_admin" >> /etc/swift/proxy-server.conf
echo "user_test_tester = testing .admin" >> /etc/swift/proxy-server.conf
echo "user_test2_tester2 = testing2 .admin" >> /etc/swift/proxy-server.conf
echo "user_test_tester3 = testing3" >> /etc/swift/proxy-server.conf


cd /etc/swift
cat >/etc/swift/swift.conf <<EOF
[swift-hash]
# random unique strings that can never change (DO NOT LOSE)
swift_hash_path_prefix = `od -t x8 -N 8 -A n </dev/random`
swift_hash_path_suffix = `od -t x8 -N 8 -A n </dev/random`
EOF

swift-ring-builder account.builder create 18 3 1
swift-ring-builder container.builder create 18 3 1
swift-ring-builder object.builder create 18 3 1

swift-ring-builder account.builder add z1-192.168.252.70:6002/loop2 10
swift-ring-builder container.builder add z1-192.168.252.70:6001/loop2 10
swift-ring-builder object.builder add z1-192.168.252.70:6000/loop2 10

swift-ring-builder account.builder add z2-192.168.252.71:6002/loop2 10
swift-ring-builder container.builder add z2-192.168.252.71:6001/loop2 10
swift-ring-builder object.builder add z2-192.168.252.71:6000/loop2 10

swift-ring-builder account.builder add z3-192.168.252.72:6002/loop2 10
swift-ring-builder container.builder add z3-192.168.252.72:6001/loop2 10
swift-ring-builder object.builder add z3-192.168.252.72:6000/loop2 10

swift-ring-builder account.builder
swift-ring-builder container.builder
swift-ring-builder object.builder

swift-ring-builder account.builder rebalance
swift-ring-builder container.builder rebalance
swift-ring-builder object.builder rebalance

chown -R swift:swift /etc/swift

sudo service swift-proxy start


PROXYSCRIPT

$objectscript = <<OBJECTSCRIPT
echo "i'm an object'" >> whoami

sh /etc/profile.d/eth2-ip.sh


apt-get install swift swift-account swift-container swift-object xfsprogs xinetd --yes
apt-get install sshpass --yes

touch /etc/rsyncd.conf
echo "uid = swift" > /etc/rsyncd.conf
echo "gid = swift" >> /etc/rsyncd.conf
echo "log file = /var/log/rsyncd.log" >> /etc/rsyncd.conf
echo "pid file = /var/run/rsyncd.pid" >> /etc/rsyncd.conf
echo "address = 192.168.252.70" >> /etc/rsyncd.conf
echo "" >> /etc/rsyncd.conf
echo "[account]" >> /etc/rsyncd.conf
echo "max connections = 2" >> /etc/rsyncd.conf
echo "path = /srv/node/" >> /etc/rsyncd.conf
echo "read only = false" >> /etc/rsyncd.conf
echo "lock file = /var/lock/account.lock" >> /etc/rsyncd.conf
echo "" >> /etc/rsyncd.conf
echo "[container]" >> /etc/rsyncd.conf
echo "max connections = 2" >> /etc/rsyncd.conf
echo "path = /srv/node/" >> /etc/rsyncd.conf
echo "read only = false" >> /etc/rsyncd.conf
echo "lock file = /var/lock/container.lock" >> /etc/rsyncd.conf
echo "" >> /etc/rsyncd.conf
echo "[object]" >> /etc/rsyncd.conf
echo "max connections = 2" >> /etc/rsyncd.conf
echo "path = /srv/node/" >> /etc/rsyncd.conf
echo "read only = false" >> /etc/rsyncd.conf
echo "lock file = /var/lock/object.lock" >> /etc/rsyncd.conf

service xinetd restart
mkdir -p /var/swift/recon

echo "[DEFAULT]" > /etc/swift/account-server.conf
echo "bind_ip = $ETH2_IP" >> /etc/swift/account-server.conf
echo "bind_port = 6002" >> /etc/swift/account-server.conf
echo "workers = 2" >> /etc/swift/account-server.conf
echo "" >> /etc/swift/account-server.conf
echo "[pipeline:main]" >> /etc/swift/account-server.conf
echo "pipeline = recon account-server" >> /etc/swift/account-server.conf
echo "" >> /etc/swift/account-server.conf
echo "[app:account-server]" >> /etc/swift/account-server.conf
echo "use = egg:swift#account" >> /etc/swift/account-server.conf
echo "" >> /etc/swift/account-server.conf
echo "[account-replicator]" >> /etc/swift/account-server.conf
echo "" >> /etc/swift/account-server.conf
echo "[account-auditor]" >> /etc/swift/account-server.conf
echo "" >> /etc/swift/account-server.conf
echo "[account-reaper]" >> /etc/swift/account-server.conf
echo "" >> /etc/swift/account-server.conf
echo "[filter:recon]" >> /etc/swift/account-server.conf
echo "use = egg:swift#recon" >> /etc/swift/account-server.conf
echo "recon_cache_path = /var/cache/swift" >> /etc/swift/account-server.conf
echo "account_recon = true " >> /etc/swift/account-server.conf

 
 
echo "[DEFAULT]" > /etc/swift/container-server.conf
echo "bind_ip = $ETH2_IP" >> /etc/swift/container-server.conf
echo "bind_port = 6001" >> /etc/swift/container-server.conf
echo "workers = 2" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[pipeline:main]" >> /etc/swift/container-server.conf
echo "pipeline = recon container-server" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[app:container-server]" >> /etc/swift/container-server.conf
echo "use = egg:swift#container" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[container-replicator]" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[container-updater]" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[container-auditor]" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[container-sync]" >> /etc/swift/container-server.conf
echo "" >> /etc/swift/container-server.conf
echo "[filter:recon]" >> /etc/swift/container-server.conf
echo "use = egg:swift#recon" >> /etc/swift/container-server.conf
echo "recon_cache_path = /var/cache/swift" >> /etc/swift/container-server.conf
echo "container_recon = true" >> /etc/swift/container-server.conf



echo "[DEFAULT]" > /etc/swift/object-server.conf
echo "bind_ip = $ETH2_IP" >> /etc/swift/object-server.conf
echo "bind_port = 6000" >> /etc/swift/object-server.conf
echo "workers = 3" >> /etc/swift/object-server.conf
echo "" >> /etc/swift/object-server.conf
echo "[pipeline:main]" >> /etc/swift/object-server.conf
echo "pipeline = recon object-server" >> /etc/swift/object-server.conf
echo "" >> /etc/swift/object-server.conf
echo "[app:object-server]" >> /etc/swift/object-server.conf
echo "use = egg:swift#object" >> /etc/swift/object-server.conf
echo "" >> /etc/swift/object-server.conf
echo "[object-replicator]" >> /etc/swift/object-server.conf
echo "" >> /etc/swift/object-server.conf
echo "[object-updater]" >> /etc/swift/object-server.conf
echo "" >> /etc/swift/object-server.conf
echo "[object-auditor]" >> /etc/swift/object-server.conf
echo "" >> /etc/swift/object-server.conf
echo "[filter:recon]" >> /etc/swift/object-server.conf
echo "use = egg:swift#recon" >> /etc/swift/object-server.conf
echo "recon_cache_path = /var/cache/swift" >> /etc/swift/object-server.conf
echo "object_recon = true" >> /etc/swift/object-server.conf




ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa

ssh-keyscan proxy1 >> /root/.ssh/known_hosts
ssh-keyscan 192.168.236.60 >> /root/.ssh/known_hosts
sshpass -p "vagrant" ssh-copy-id proxy1


scp proxy1:/etc/swift/*.ring.gz /etc/swift/

scp proxy1:/etc/swift/swift.conf /etc/swift/

chown -R swift:swift /etc/swift

mkdir -p /srv/node/loop2


dd if=/dev/zero of=/mnt/object-volume1 bs=1 count=0 seek=10G
losetup /dev/loop2 /mnt/object-volume1

mkfs.xfs -i size=1024 /dev/loop2
echo "/dev/loop2 /srv/node/loop2 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0" >> /etc/fstab
mount -a

chown swift:swift /srv/node/loop2

service swift-account restart; service swift-container restart; service swift-container-updater restart; service swift-object-updater restart; service swift-account-auditor restart; service swift-container-auditor restart; service swift-object restart; service swift-account-reaper restart; service swift-container-replicator restart; service swift-object-auditor restart; service swift-account-replicator restart; service swift-container-sync restart; service swift-object-replicator restart; 



OBJECTSCRIPT

Vagrant.configure(2) do |config|

  config.vm.box = "ubuntu/trusty64"
  

  # Turn off shared folders
  config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true

  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]

      config.vm.provision :shell, inline: $commonscript

      config.vm.network :private_network, ip: opts[:eth1]
      config.vm.network :private_network, ip: opts[:eth2]

      config.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"] = opts[:mem]
        v.vmx["numvcpus"] = opts[:cpu]
      end

      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
      end

      if opts[:nodetype] == "proxy"
          config.vm.provision :shell, inline: $proxyscript
      end

      if opts[:nodetype] == "object"
          config.vm.provision :shell, inline: $objectscript
      end
    end
  end
end
