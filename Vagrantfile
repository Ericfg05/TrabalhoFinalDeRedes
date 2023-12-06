Vagrant.configure("2") do |config| 

  config.vm.box = "gusztavvargadr/ubuntu-server"    #1 - Box (imagem base) a ser usada por todas as vms

#    config.vm.define "vm1" do |vm1| #dhcp
  
#      vm1.vm.network "forwarded_port", guest: 67, host: 3000  #2  - Configuração de rede
#      vm1.vm.network "private_network", ip: "192.168.56.10"
#      vm1.vm.synced_folder "./dhcpd", "/config"

#     vm1.vm.provision "shell", inline: <<-SHELL
#      export DEBIAN_FRONTEND=noninteractive
#      apt update
#      apt install -y docker.io
#      chmod 777 /config
#      cd /config
#      docker run --privileged -d --name dhcp --net host -p 67:67 -e "IP=172.21.0.100" \
#       homeall/dhcphelper:latest -v /config/udhcpd.conf:/etc 
     

#  SHELL
  #  end

  #  config.vm.define "vm2" do |vm2| #dns

  #    vm2.vm.network "forwarded_port", guest: 53, host: 5352  #2  - Configuração de rede
  #    vm2.vm.network "private_network", ip: "192.168.56.11"
  #    vm2.vm.synced_folder "/var/www/html", "/var/www/html" #2 - Pasta Compartilhada
   
  #    vm2.vm.provision "shell", inline: <<-SHELL
  
    
  #      export DEBIAN_FRONTEND=noninteractive
  #      apt update
  #      apt install -y docker.io
  #      docker run -d --name bind9-container -e TZ=UTC -p 30053:53 ubuntu/bind9:9.18-22.04_beta
        
  #    SHELL
  #  end

  #    config.vm.define "vm3" do |vm3|
  
  #    vm3.vm.network "private_network", ip: "192.168.56.13"
  #    vm3.vm.synced_folder "./diretorioPC", "/diretorioVM1" 
  #    vm3.vm.network "forwarded_port", guest: 81, host: 8081  #2  - Configuração de rede

  #    vm3.vm.provision "shell", inline: <<-SHELL

  #    apt update
  #    apt install -y docker.io
  #    chmod 777 /diretorioVM1
  #    cd /diretorioVM1
  #    wget google.com
  #    docker run -d --name serverweb4P -v /diretorioVM1:/usr/local/apache2/htdocs/ -p 80:80 httpd 
    
  #    SHELL
  #  end
  # config.vm.define "vm4" do |vm4| #ftp
  #   vm4.vm.network "private_network", ip: "192.168.56.13"
    
  #   vm4.vm.network "forwarded_port", guest: 21, host:21  
  #   vm4.vm.network "forwarded_port", guest: 47400, host:47400
  #   vm4.vm.network "forwarded_port", guest: 20, host:20
  #   vm4.vm.network "forwarded_port", guest: 82, host:8082
  #   vm4.vm.synced_folder "./ftp", "/ftp" 
   
   
  #   vm4.vm.provision "shell", inline: <<-SHELL

  #   apt update
  #   chmod 777 /ftp
  #   cd /ftp
  #   apt install -y docker.io
  #   docker run -d -v /ftp:/home/vsftpd -p 20:20 -p 21:21 -p 47400:47400 -e FTP_USER=ekode -e FTP_PASS=ekode123 -e PASV_ADDRESS=10.0.75.1 --name ftp --restart=always bogem/ftp
  #   SHELL
  #   end
     config.vm.define "vm5" do |vm5|#nfs

       vm5.vm.network "forwarded_port", guest: 1024, host:1024  
     
       vm5.vm.synced_folder "./ftp", "/nfs" 
     
        vm5.vm.provision "shell", inline: <<-SHELL
         export DEBIAN_FRONTEND=noninteractive
         apt update
         chmod 777 /nfs
         cd /nfs
         apt install -y docker.io
        
        
         docker run                                            \
            -v /nfs:/home \
            -v /nfs:/etc/exports  \
           --cap-add SYS_ADMIN                                 \
           -p 1024:1024                                        \
           erichough/nfs-server
    
        SHELL
       end
end