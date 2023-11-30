Vagrant.configure("2") do |config|

  config.vm.box = "gusztavvargadr/ubuntu-server"   # 1 - Box (imagem base) a ser usada por todas as vms

  config.vm.define "vm1" do |vm1|
  
    vm1.vm.network "forwarded_port", guest: 67, host: 67 # 2  - Configuração de rede
    vm1.vm.network "private_network", ip: "192.168.56.10"
  
 
  vm1.vm.provision "shell", inline: <<-SHELL
    #ip route add default via 192.168.56.12
    export DEBIAN_FRONTEND=noninteractive
    apt update
    apt install -y docker.io
    docker run --privileged -d --name dhcp --net host -e "IP=172.21.0.100" homeall/dhcphelper:latest
    
SHELL
  end

  config.vm.define "vm2" do |vm2|

    vm2.vm.network "forwarded_port", guest: 52, host: 52 # 2  - Configuração de rede
    vm2.vm.network "private_network", ip: "192.168.56.11"
    vm2.vm.synced_folder "/var/www/html", "/var/www/html" #2 - Pasta Compartilhada
   
    vm2.vm.provision "shell", inline: <<-SHELL
     
      export DEBIAN_FRONTEND=noninteractive
      apt update
      apt install -y docker.io
      docker run --name dns -d -e DNS_DOMAIN=docksal -e DNS_IP=192.168.56.11 docksal/dns
        
    SHELL
  end

  config.vm.define "vm3" do |vm3|

  vm3.vm.network "forwarded_port", guest: 90, host: 90 # 
 
  vm3.vm.synced_folder "/var/www/html", "/var/www/html" 
 
   vm3.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt update
    apt-get install -y apache2
   # sudo nano /etc/exports
    apt install -y docker.io
    docker run -itd --restart=always \
    -p 20-21:20-21 \
    -p 82:82 \
    -p 40000-40050:40000-40050 \
    -v $LOCAL_DIR/data:/srv/ftp \
    -v $LOCAL_DIR/log:/var/log \
    -v $LOCAL_DIR/home:/home \
    -e PRIVATE_PASSWD=secret \
    -e PASV_ADDRESS=$PUBLIC_IP_ADDRESS \
    ustclug/ftp

    docker run                                            \
       -v /var/www/html:/home \
       -v /var/www/html/exports.txt:/etc/exports:ro  \
      --cap-add SYS_ADMIN                                 \
      -p 2049:2049                                        \
      erichough/nfs-server

   SHELL
  end
end