# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']

MACHINES = {
  
  :ovclient => {
        :box_name => "ubuntu/xenial64",
        :ip_addr => '192.168.11.151',
  } ,
  :ovserver => {
        :box_name => "ubuntu/xenial64",
        :ip_addr => '192.168.11.150',
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "1024"]
          #vb.customize ["modifyvm", :id, "--cpus", "2"]

        #   vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
          vb.name = boxname.to_s

        #   boxconfig[:disks].each do |dname, dconf|
        #       unless File.exist?(dconf[:dfile])
        #         vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
        #       end
        #       vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
        # end
          end
      box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL
          
      case boxname.to_s
      
      when "ovclient"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
        echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCW+VHI6di+7jZZhnYiCUciVO3oCSJ1xkV+8TINsNy1Itek0BUnorH+Mh6wC5eHoFVsid39v5A5ypzYZvJWhjwu4LNBJFroNhPnpmSBoA7Xk9U+slDI1A6pImop3qQbncMbYMdeyK5yoQO9bgJKDoQG7ak99qp24C4koFHGXO9Bejhenkkct2j0iTQreRyv2y3oSeOvsvQcBFuYS3H0FPhTUII8dx+/tjOTYFaxiA+EkWhuyXfhnrUd60BN5+ajqEgtv4CYZm2MBzDWu3Sor142Ms3R/FbwF1MJKd7JHOzJcTARfnpBqBZi+Or+l9+Pdl8yzxbxO0+9yaj7MGP9eyVT" >> /home/vagrant/.ssh/authorized_keys
        echo "192.168.11.150  ovserver" >> /etc/hosts
        sudo cp /vagrant/id_rsa /home/vagrant/.ssh/
        sudo chown vagrant:vagrant /home/vagrant/.ssh/id_rsa 
        sudo chmod 0600 /home/vagrant/.ssh/id_rsa
        sudo apt update
        sudo apt-get install ansible vim python-pip python-apt -y
        pip install pyOpenSSL
        SHELL
      when "ovserver"
      
       

        config.vm.provision :file do |file|
          file.source = "./openvpn/"
          file.destination = "/home/vagrant/roles/"
        end
        
        
        config.vm.provision :file do |file|
          file.source = "./openvpn-server-client.yml"
          file.destination = "/home/vagrant/openvpn-server-client.yml"
        end
        config.vm.provision :file do |file|
          file.source = "./openvpn-client.yml"
          file.destination = "/home/vagrant/openvpn-client.yml"
        end
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          
          sudo apt-add-repository ppa:ansible/ansible
          sudo apt update
          sudo apt-get install ansible vim python-pip python-apt -y
          sudo pip install netaddr
          sudo cp /vagrant/id_rsa /home/vagrant/.ssh/
          sudo chown vagrant:vagrant /home/vagrant/.ssh/id_rsa 
          sudo chmod 0600 /home/vagrant/.ssh/id_rsa
          sudo echo "192.168.11.151  ovclient" >> /etc/hosts
          sudo echo "192.168.11.150  ovserver" >> /etc/hosts
          sudo echo -e "[openvpn-clients] \n  ovclient ansible_host=ovclient ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa \n [openvpn-servers] \n  ovserver ansible_host=ovserver ansible_ssh_private_key_file=/home/vagrant/.ssh/id_rsa" > /etc/ansible/hosts
          
          sudo echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCW+VHI6di+7jZZhnYiCUciVO3oCSJ1xkV+8TINsNy1Itek0BUnorH+Mh6wC5eHoFVsid39v5A5ypzYZvJWhjwu4LNBJFroNhPnpmSBoA7Xk9U+slDI1A6pImop3qQbncMbYMdeyK5yoQO9bgJKDoQG7ak99qp24C4koFHGXO9Bejhenkkct2j0iTQreRyv2y3oSeOvsvQcBFuYS3H0FPhTUII8dx+/tjOTYFaxiA+EkWhuyXfhnrUd60BN5+ajqEgtv4CYZm2MBzDWu3Sor142Ms3R/FbwF1MJKd7JHOzJcTARfnpBqBZi+Or+l9+Pdl8yzxbxO0+9yaj7MGP9eyVT" >> /home/vagrant/.ssh/authorized_keys
          # Create project structure 
          sudo mkdir ~/ansible
          sudo sed -i 's/#host_key_checking = False/host_key_checking = False/g' /etc/ansible/ansible.cfg
          export ANSIBLE_HOST_KEY_CHECKING=False
          ansible-playbook openvpn-server-client.yml 
          #ansible-playbook openvpn-client.yml 
          SHELL
          
      
      end

      end
   end
end
