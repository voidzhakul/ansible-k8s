$install_ansible= <<SCRIPT
yum  install -y wget
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum makecache
yum install -y ansible
SCRIPT

$change_ssh=<<SCRIPT
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd
SCRIPT

$init_ansible=<<SCRIPT
ip=192.168.92.

PASSWORD=vagrant

ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
sed -i '/StrictHostKeyChecking/c StrictHostKeyChecking no' /etc/ssh/ssh_config

rpm -q sshpass  || yum -y install sshpass
echo [master] >> /etc/ansible/hosts 
for i in {0..3}
do
    if [ $i == 1 ]
    then
        echo [node] >> /etc/ansible/hosts 
    fi
    echo "${ip}$[i+100] ansible_ssh_user=root" >> /etc/ansible/hosts 
    sshpass -p $PASSWORD ssh-copy-id -i  root@${ip}$[i+100]  
done
SCRIPT

Vagrant.configure("2") do |config|
    (1..3).each do |i|
        config.vm.define "node#{i}" do |node|
            node.vm.box = "centos/7"
            node.vm.hostname="node#{i}"
            node.vm.network "private_network", ip: "192.168.92.#{100+i}"
            node.vm.provider "virtualbox" do |v|
                v.name = "node#{i}"
                v.memory = 2048
                v.cpus = 1
            end
            node.vm.  provision "shell",inline: $change_ssh
        end
    end

    config.vm.define "master" do |master|
        master.vm.box = "centos/7" 
        master.vm.hostname = "master"
        master.vm.network "private_network",ip: "192.168.92.100"
        master.vm.provider "virtualbox" do |v|
            v.name = "master"
            v.memory =2048
            v.cpus = 2
        end

        master.vm.provision "shell", inline: $install_ansible
        master.vm.provision "shell", inline: $change_ssh
        master.vm.provision "shell",inline: $init_ansible
        
    end
end
