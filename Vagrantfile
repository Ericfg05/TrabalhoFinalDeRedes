Vagrant.configure("2") do |config|

  config.vm.box = "gusztavvargadr/ubuntu-server"   # 1 - Box (imagem base) a ser usada por todas as vms

  config.vm.define "vm1" do |vm1|

    vm1.vm.network "forwarded_port", guest: 67, host: 67 # 2  - Configuração de rede
    vm1.vm.network "private_network", ip: "192.168.56.10"
    vm1.vm.synced_folder "/var/www/html", "/var/www/html" #3 - Pasta Compartilhada
   
    vm1.vm.provision "shell", inline: <<-SHELL
      #ip route add default via 192.168.56.12
      export DEBIAN_FRONTEND=noninteractive
      apt update
      apt install -y docker.io
      #docker build -t gns3/dhcp .
      #docker run -dit --name gns3/dhcp
      #docker push gns3/dhcp
    SHELL
  end

  config.vm.define "vm2" do |vm2|

    vm2.vm.network "forwarded_port", guest: 53, host: 53 # 2  - Configuração de rede
    vm2.vm.network "private_network", ip: "192.168.56.11"
    vm2.vm.synced_folder "/var/www/html", "/var/www/html" #3 - Pasta Compartilhada
   
    vm2.vm.provision "shell", inline: <<-SHELL
      #ip route add default via 192.168.56.12
      export DEBIAN_FRONTEND=noninteractive
      apt update
      apt install -y docker.io
      docker run --name dns -d -e DNS_DOMAIN=docksal -e DNS_IP=192.168.56.11 docksal/dns
    SHELL
  end
  config.vm.define "vm3" do |vm3|

    vm3.vm.network "forwarded_port", guest: 84, host: 1234 # 2  - Configuração de rede
    vm3.vm.network "private_network", ip: "192.168.56.11"
    vm3.vm.synced_folder "/var/www/html", "/var/www/html" #3 - Pasta Compartilhada
   
    vm3.vm.provision "shell", inline: <<-SHELL
      #ip route add default via 192.168.56.12
      export DEBIAN_FRONTEND=noninteractive
      apt update
      apt install -y docker.io
      docker run -d --name apache2-container -e TZ=UTC -p 8080:80 ubuntu/apache2:2.4-22.04_beta
      docker run -d --name vsftpd -p 32021:21 -p 32022-32041:32022-32041 -v /var/lib/ftp/incoming vsftpd-anonymous-upload
      docker run -P -p 2049:2049 --name ceph-nfs \ 
      -e 'CEPH_CLIENT_ID=ganesha' \
      -e 'CEPH_CLIENT_KEY=AbC123==' \
      -v /etc/krb5.conf:/etc/krb5.conf:ro \
      -v /etc/krb5.keytab:/etc/krb5.keytab:ro \
      -v/etc/sssd/sssd.conf:/etc/sssd/sssd.conf:ro \
      --hostname=nfs.example.edu --privileged miosiris/nfs-ganesha-ceph
      

    SHELL
  end

end
