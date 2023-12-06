**Requisitos**

Para realizar esse tipo de configuração de redes, são necessários possuir alguns softwares, como, Vagrant, Docker e VirtualBox. Todos eles instalados no sistema operacional Linux.

**Instalação**

Para iniciar a com a utilização do Vagrant, você deve primeiramente baixar a ferramenta, para isso, abra o terminal (Ctrl+Alt+t), e digite o seguinte comando: sudo apt- get update, assim esse comando fará as atualizações de dependências em seu sistema, após atualizar, execute o comando: sudo apt install vagrant , assim baixará a ferramenta vagrant. O vagrant utiliza-se o VirtualBox, para criação das máquinas virtuais, para download do VirtualBox, utilize o seguinte comando em seu terminal: sudo apt install virtualbox-qt, assim a ferramenta será instalada em seu computador. O docker que iremos utilizar para criação dos containers será instalado dentro da máquina virtual. Porém caso deseje instalar em sua máquina host utilize o comando sudo apt install -y docker-io, esse comando será também utilizado dentro das máquinas virtuais para a instalação do docker. Para instalar o visual code,  entre no site oficial e baixe o pacote .deb, logo realizar o Download, entre na pasta Download  e abra o terminal, e digite o comando sudo dpkg -i nome_do_arquivo assim, instalando-o em sua máquina

**Inicialização do Vagrant**

Para iniciar o Vagrant , você deve utilizar uma imagem de um sistema operacional, assim você poderá criar uma máquina virtual de qualquer sistema. As imagens são disponibilizadas no site oficial: https://app.vagrantup.com/boxes/search , porém, para a criação das máquinas, será utilizado o ubuntu server. Primeiramente, abre o terminal (Ctrl+Alt+t), e em seguida crie uma pasta com o nome que você deseja, use o comando: mkdir nome_da_pasta , em seguida entre na pasta com o comando cd nome_da_pasta, em sequência, use o comando: vagrant init gusztavvargadr/ubuntu-server, com isso será criado um arquivo com o nome Vagrantfile, assim basta dar o comando code . para abrir e editar o arquivo no visual studio.

**Requisitos dos serviços** 

|Requisitos|Descrição                                                                                                       |
|---   | --------------------------------------------------------------------------------------------------------------------- |
| RQ01 | Configure um servidor DHCP no ambiente Linux para atribuir endereços IP automaticamente aos dispositivos na rede.  |
| RQ02 | Implante um servidor DNS para resolver nomes de domínio dentro da rede e configurar registros DNS como A, CNAME, MX |
| RQ03 | Configure e hospede um servidor web Apache ou Nginx para fornecer serviços de hospedagem de sites internos          |
| RQ04 | Implemente um servidor FTP (por exemplo, vsftpd) para permitir a transferência de arquivos na rede                  |
| RQ05 | Configure um servidor NFS para compartilhar diretórios e arquivos entre máquinas na rede.                           |



**Topologia utilizada**

A topologia desse projeto consistiu em um computador (HOST)  fornecendo internet e hardware para as VMS, assim utilizando o Vagrant e o Virtualbox para criar as máquinas onde serão criadas as 3 VMS previstas onde  elas devem se comunicar entre si.
A topologia pode ser  visualizada na imagem abaixo: 
![ TrabalhoFinalDeRedes
/Topologia.png
](Topologia.png)




**Método de construção**

Para adicionar os requisitos pedidos, é necessário estruturar seu Vagrantfile. A estrutura base que será utilizado: 

Vagrant.configure("2") do |config|

 config.vm.box = "gusztavvargadr/ubuntu-server"   # 1 - Box (imagem base) a ser usada por todas as vms

 config.vm.define "vm1" do |vm1|

   vm1.vm.network "forwarded_port", guest: 67, host: 67 # 2  - Configuração de rede
   vm1.vm.network "private_network", ip: "192.168.56.10"
 
   vm1.vm.provision "shell", inline: <<-SHELL

     export DEBIAN_FRONTEND=noninteractive
     apt update
     apt install -y docker.io

   SHELL
 end
Esse código será base para todas as vms que serão criadas, serão ao todo três vms até o momento. Sendo assim, códigos que devem ser adicionado para rodar as imagens da requisição são os docker run como por exemplo docker run --name dns -d -e DNS_DOMAIN=docksal -e DNS_IP=192.168.56, que cria uma imagem DNS, que seria o requisito da VM2. Também podemos observar docker run -d --name apache2-container -e TZ=UTC -p 8080:80 ubuntu/apache2:2.4-22.04_beta, que assim cria uma imagem do apache (servidor web da VM3). Lembrando que esses comandos devem estar dentro do <<- SHELL seguindo o exemplo abaixo:
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
A imagens podem ser extraídas pelo site dock hub: https://hub.docker.com/

**Teste**

Para realizar o teste do funcionamento, pode se inicializar o terminal entrar na pasta em que o Vagrantfile foi criado, e em seguida nesta pasta e digite o comando vagrant ssh nome da vm, exemplo: vagrant ssh vm2, assim você entrará dentro da vm, assim você pode dar um sudo docker ps, dessa maneira, será listados todos os conteiner criados. 
Para testar o DHCP  use o dhcp.config para poder visualizar a configuração do DHCP.
Para acessar os containers use o comando sudo docker exec -it nome do container /bin/bash, assim pode usar o comando apache2 -v para verificar se o apache foi instalado, caso sim ele retornara sua versão.
Para testar a funcionalidade do FTP, entre dentro do container, e use o comando status, assim ele retornará o status atual da conexão.




