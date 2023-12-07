**Requisitos**

Para realizar esse tipo de configuração de redes, são necessários possuir alguns softwares, como, Vagrant, Docker e VirtualBox. Todos eles instalados no sistema operacional Linux.

**Instalação**

Para iniciar a com a utilização do Vagrant, você deve primeiramente baixar a ferramenta, para isso, abra o terminal (Ctrl+Alt+t), e digite o seguinte comando: sudo apt- get update, assim esse comando fará as atualizações de dependências em seu sistema, após atualizar, execute o comando: **sudo apt install vagrant** , assim baixará a ferramenta vagrant. O vagrant utiliza-se o VirtualBox, para criação das máquinas virtuais, para download do VirtualBox, utilize o seguinte comando em seu terminal: **sudo apt install virtualbox-qt**, assim a ferramenta será instalada em seu computador. O docker que iremos utilizar para criação dos containers será instalado dentro da máquina virtual. Porém, caso deseje instalar em sua máquina host utilize o comando **sudo apt install -y docker-io**, esse comando será também utilizado dentro das máquinas virtuais para a instalação do docker. Para instalar o visual code,  entre no site oficial e baixe o pacote .deb, logo que realizar o Download, entre na pasta Download e abra o terminal, e digite o comando: **sudo dpkg -i nome_do_arquivo**, assim, instalando-o em seu computador

**Inicialização do Vagrant**

Para iniciar o Vagrant , você deve utilizar uma imagem de um sistema operacional, assim você poderá criar uma máquina virtual de qualquer sistema. As imagens são disponibilizadas no site oficial: **https://app.vagrantup.com/boxes/search** , porém, para a criação das máquinas, será utilizado o ubuntu server **(gusztavvargadr/ubuntu-server)**. Primeiramente, abre o terminal (Ctrl+Alt+t), e em seguida crie uma pasta com o nome que você deseja, use o comando: **mkdir nome_da_pasta** para isso, em seguida entre na pasta com o comando **cd nome_da_pasta**, em sequência, use o comando: **vagrant init gusztavvargadr/ubuntu-server**, com isso será criado um arquivo com o nome Vagrantfile, assim basta dar o comando **code .** para abrir e editar o arquivo no visual studio.

**Requisitos dos serviços** 

|Requisitos|Descrição                                                                                                       |
| ---  | --------------------------------------------------------------------------------------------------------------------- |
| RQ01 | Configure um servidor DHCP no ambiente Linux para atribuir endereços IP automaticamente aos dispositivos na rede.  |
| RQ02 | Implante um servidor DNS para resolver nomes de domínio dentro da rede e configurar registros DNS como A, CNAME, MX |
| RQ03 | Configure e hospede um servidor web Apache ou Nginx para fornecer serviços de hospedagem de sites internos          |
| RQ04 | Implemente um servidor FTP (por exemplo, vsftpd) para permitir a transferência de arquivos na rede                  |
| RQ05 | Configure um servidor NFS para compartilhar diretórios e arquivos entre máquinas na rede.                           |



**Topologia utilizada**

A topologia desse projeto consistiu em um computador (HOST)  fornecendo internet e hardware para as VMS, assim utilizando o Vagrant e o Virtualbox para criar as máquinas virtuais. Ao todo será criado 5 máquinas virtuais, que pode ser observado na topologia abaixo.
A topologia pode ser  visualizada na imagem abaixo: 
![ TrabalhoFinalDeRedes
/Topologia.png
](Topologia.png)

Cada VM possuem um container via docker, que possui um tipo de serviço rodando, pode-se observar na tabela abaixo o tipo de seviço que as vm rodam é o nome da imagem utilizada.
| Maquinas virtuais | Tipo de serviço | Nome da Imagem (DOCKER)
| --- | --- | --- |
| VM1 | Serviço de DHCP | homeall/dhcphelper:latest |
| VM2 | Serviço de DNS | ubuntu/bind9:9.18-22.04_beta |
| VM3 | Serviço web APACHE 2 | httpd |
| VM4 | Serviço FTP | bogem/ftp |
| VM5 | Serviço NFS| erichough/nfs-server |



**Método de construção**

Para adicionar os requisitos pedidos, é necessário estruturar seu Vagrantfile. A estrutura base que será utilizada pode ser visto abaixo: 

Vagrant.configure("2") do |config|

 config.vm.box = "gusztavvargadr/ubuntu-server"   # 1 - Box (imagem base) a ser usada por todas as vms

 config.vm.define "vm1" do |vm1|

   vm1.vm.network "forwarded_port", guest: numero_da_porta_convidada, host: numero_da_porta_host # 2  - Configuração de rede
 
   vm1.vm.provision "shell", inline: <<-SHELL

   export DEBIAN_FRONTEND=noninteractive   
   apt update
   apt install -y docker.io

   SHELL
 end
Esse código será base para todas as vms que serão criadas. Sendo assim, códigos que devem ser adicionado para rodar as imagens da requisição são os docker run.

|Serviço | Comando docker run|
|---|---|
| dhcp |docker run --privileged -d --name dhcp --net host -p 67:67 -e "IP=172.21.0.100" homeall/dhcphelper:latest -v /config/udhcpd.conf:/etc
| DNS | docker run -d --name bind9-container -e TZ=UTC -p 30053:53 ubuntu/bind9:9.18-22.04_beta |
| APACHE2 | docker run -d --name serverweb4P -v /diretorioVM1:/usr/local/apache2/htdocs/ -p 80:80 httpd |
| FTP |docker run -d -v /ftp:/home/vsftpd -p 20:20 -p 21:21 -p 47400:47400 -e FTP_USER=ekode -e FTP_PASS=ekode123 -e PASV_ADDRESS=10.0.75.1 --name ftp --restart=always bogem/ftp|
| NFS |  -v /nfs:/home -v /nfs:/etc/exports --cap-add SYS_ADMIN -p 1024:1024 erichough/nfs-server |

**Lembrando que esses comandos devem estar dentro do <<- SHELL** seguindo o exemplo abaixo:
config.vm.define "vm2" do |vm2|

   vm2.vm.network "forwarded_port", guest: 80, host: 8080 # 2  - Configuração de rede
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
**A imagens podem ser extraídas pelo site dock hub: https://hub.docker.com/**

**Teste**

***Teste geral***
Nesse item, descreve a forma geral padrão de teste de cada vm.

Para realizar o teste do funcionamento, pode se inicializar o terminal, entrando na pasta em que o Vagrantfile foi criado, e em seguida nesta pasta e digite o comando vagrant ssh nome da vm, exemplo: vagrant ssh vm2, assim você entrará dentro da vm, posteriormente você pode usar o comando sudo docker ps, dessa maneira, será listados todos os conteiner criados. E para finalizar use o comando sudo docker exec -it id_container, assim entrando dentro do container. obs: a VM1 troque o /bin/bash por /bin/sh para entrar no container

***Teste especifico***
|VMS|Testes
| --- | --- |
|VM1| Para testar se o DHCP foi criado corretamente, após seguir os passa passo do item anterior, use o comando cat /etc e após use o comando cat /udhcp.conf, assim abrirá o arquivo e msotrará a configuração, pode ser visto na imagem 1 |
|VM3|| Para testar o funcionamento do apache, entre no container conforme o passo a passo ***teste geral***, em seguida entre na pasta var/www/html e verá o arquivo HTML compartilhado.
| VM4 | Para testar o FTP siga o passo a passo do item ***teste geral*** em seguida use o comando cd /home/vsftpd e depois cat vsftp.config  para visualizar o arquivo |

![ TrabalhoFinalDeRedes
/DHCP.png
](DHCP.png)

Para acessar os containers use o comando sudo docker exec -it nome do container /bin/bash, assim pode usar o comando apache2 -v para verificar se o apache foi instalado, caso sim ele retornara sua versão, em seguida procure a pasta em que foi compartilhado o site. Para melhor visualização segue a imagem abaixo:

