# asterisk_docker
Repositório com tutorial para instalação do Asterisk 18.15.0 num container Docker.

[![Vagrant](https://img.shields.io/badge/Vagrant-2C2D72?style=for-the-badge&logo=vagrant)](https://www.vagrantup.com/)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox)](https://www.virtualbox.org/)
[![Asterisk](https://img.shields.io/badge/Asterisk-0091C8?style=for-the-badge&logo=asterisk)](https://www.asterisk.org/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker)](https://www.docker.com/)

## Criação e configuração do ambiente virtualizado:

**Requisitos:**
- [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://developer.hashicorp.com/vagrant/downloads)

Para criar o ambiente onde instalaremos o Asterisk em um container Docker, utilizaremos o Virtualbox, um software de virtualização que permite criar e executar máquinas virtuais em seu computador, e o Vagrant, uma ferramenta que permite criar e gerenciar ambientes de desenvolvimento virtualizados. Ambos estão disponíveis em várias plataformas e podem ser baixados pelos links fornecidos na seção de requisitos.

Após a instalação, siga para a próxima etapa, que é a criação do ambiente virtual para o provisionamento do Asterisk em um container Docker. Neste tutorial, usaremos o CentOS 7 como sistema base, uma distribuição Linux de código aberto baseada no Red Hat Enterprise Linux (RHEL).

Para provisionar a máquina virtual, verifique se os requisitos estão instalados no <b>sistema hospedeiro</b> utilizando os seguintes comandos:

```bash
$ virtualbox --version
$ vagrant --version
```

Criaremos uma pasta e dentro dela um novo arquivo com o nome Vagrantfile. Dentro dele inserimos os seguintes comandos:

```ruby
Vagrant.configure("2") do |asterisk|
  asterisk.vm.box = "centos/7"
  asterisk.vm.hostname = "asterisk"
  asterisk.vm.network "public_network"
  asterisk.vm.provider "virtualbox" do |vb|
    vb.name = "Asterisk-Docker"
    vb.memory = "2048"
    vb.cpus = "2"
  end
  asterisk.vm.provision "shell", inline: <<-SHELL
  sudo yum update -y
  sudo yum install curl git vim make wget -y
  SHELL
end
```

Esta configuração do arquivo Vagrantfile define o provisionamento no Virtualbox de uma máquina virtual com 2 processadores e 2 GB de memória ram, sistema operacional CentOS 7 atualizado e instalação dos softwares curl, git, vim, make e wget que serão necessários para o propósito do trabalho. Caso prefira o Vagrantfile pode ser baixado através do link https://raw.githubusercontent.com/zeraimundo/asterisk_docker/main/Vagrantfile.

Com o Vagrantfile criado abriremos na pasta um terminal e executaremos o comando “vagrant up”, ele usará o Vagrant para ler o arquivo previamente configurado e fará o provisionamento necessário para continuação do trabalho.

Concluído o provisionamento do sistema, faremos acesso ao mesmo através do comando “vagrant ssh”, em seguida faremos a instalação do Docker, uma plataforma de virtualização de contêineres de software e que será utilizada para implementarmos o Asterisk.

Instalaremos o Docker seguindo as informações do site da desenvolvedora - https://docs.docker.com/engine/install/ . Para facilitar o processo, disponibilizamos os comandos para instalação no sistema CentOS.
Primeiro adicionaremos o repositório do Docker no CentOS 7:

```bash
$ sudo yum install -y yum-utils
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Em seguida instalaremos a ultima versão disponível do Docker:

```bash
$ sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/d8e9c685-3a60-4bd2-bd8d-6396db403b59)


Agora instalaremos o docker-compose, uma ferramenta que permite definir e gerenciar aplicativos multi-contêineres em um único arquivo de configuração e será necessário para o nosso trabalho.

Para tal utilizaremos os seguintes comandos que sequencialmente instalarão o aplicativo, darão a ele permissão de execução e o moverão para a pasta /usr/local/bin/:

```bash
$ curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
$ chmod +x docker-compose-linux-x86_64
$ sudo mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```

Verificaremos a instalação com o comando “docker-compose --version”. 

```bash
$ docker-compose --version
```
![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/071e2f66-51e2-4c12-8f38-7e45a4e73ac9)

Agora adicionaremos o usuário atual ao grupo Docker:

```bash
$ sudo usermod -aG docker $USER
$ newgrp docker
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/b8673e8f-1fab-4cdd-a378-553cfc911a89)

Por fim, configuramos a inicialização do Docker:

```bash
$ sudo systemctl start docker && sudo systemctl enable docker
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/e96de383-ec3e-466d-8d57-d81f5bd52c1d)

Com isso teremos sistema operacional instalado e pronto para receber as configurações para instalarmos o Asterisk num container Docker.

## Configurando o contêiner Asterisk:


Agora faremos as configurações para implementar o contêiner Docker. Utilizaremos para tal o repositório mlan/Asterisk que pode ser encontrado em:  https://github.com/mlan/docker-asterisk .

Faremos o clone do repositório, alteraremos o seu nome de demo para deploy e acessaremos sua pasta com os seguintes comandos:

```bash
$ git clone https://github.com/mlan/docker-asterisk.git
$ cd docker-asterisk
$ mv demo deploy
$ cd deploy
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/1b58c524-1bc9-456d-b0ca-88bc01e19f5b)

## Criar um armazenamento persistente para o Asterisk:

No nosso projeto o Asterisk será instalado em um container Docker, para facilitar sua configuração faremos a criação de um armazenamento local. Desta forma poderemos acessar a pasta com as configurações do Asterisk diretamente no nosso sistema hospedeiro.

Inicialmente criaremos a pasta para o armazenamento e alteraremos suas permissões utilizando os seguintes comandos:

```bash
$ sudo mkdir -p /data/asterisk-conf
$ sudo chmod 775 -R /data/asterisk-conf
$ sudo chown -R $USER:docker /data/asterisk-conf
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/83f78b29-f1c2-4b6f-b12c-9d9c048784cd)

Como estamos utilizando o sistema CentOS 7 para permitir essa configuração se faz necessária a alteração do SELinux, mecanismo de segurança de controle de acesso obrigatório projetado para fornecer uma camada adicional de segurança no sistema operacional Linux.

Utilizaremos o seguinte comando “sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config”  que desativa permanetemente a aplicação das políticas de segurança do SELinux, colocando-o no modo "Permissive".

```bash
$ sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/e12d5dc8-ab40-456b-9b78-64d6460a3f5a)

Configurar o SELinux como "permissive" significa definir o modo de execução do SELinux para permitir ações que normalmente seriam bloqueadas pelas políticas de segurança do SELinux, mas ainda registrar essas violações como avisos.

Agora que temos permissão do sistema poderemos criar o volume para o armazenamento persistente do nosso container utilizando o comando:

```bash
$ docker volume create --driver local \
     --opt type=none \
     --opt device=/data/asterisk-conf \
     --opt o=bind asterisk-conf
```

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/465d5326-09bf-4097-933a-aa227c9c6208)

Podemos confirmar a criação através do comando:

```bash
$ docker volume list
```
![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/5f0a94aa-f45b-455c-b9c8-dc8ab6b4be70)

Com nosso armazenamento criado podemos agora configurar o arquivo docker-compose.yml. Ele está localizado no diretório onde clonamos o repositório, dentro da pasta deploy.

Para edita-lo usaremos o vim, através do comando:

```bash
$ vim docker-compose.yml
```
```ruby
version: '3'

services:
  tele:
    image: mlan/asterisk
    network_mode: bridge                    # Only here to help testing
    cap_add:
      - sys_ptrace                          # Only here to help testing
      - net_admin                           # Allow NFT, used by AutoBan
      - net_raw                             # Allow NFT, used by AutoBan
    ports:
      - "${SMS_PORT-8080}:${WEBSMSD_PORT:-80}" # WEBSMSD port mapping
      - "5060:5060/udp"                     # SIP UDP port
      - "5060:5060"                         # SIP TCP port
      - "5061:5061"                         # SIP TLS port
      - "10000-10099:10000-10099/udp"       # RTP ports
    environment:
      - SYSLOG_LEVEL=${SYSLOG_LEVEL-4}      # Logging
      - HOSTNAME=${TELE_SRV-tele}.${DOMAIN-docker.localhost}
      - PULSE_SERVER=unix:/run/pulse/socket # Use host audio
      - PULSE_COOKIE=/run/pulse/cookie      # Use host audio
      - WEBSMSD_PORT=${WEBSMSD_PORT-80}     # WEBSMSD internal port
    volumes:
      - asterisk-conf:/srv                      # Persistent storage
      - ./pulse:/run/pulse:rshared          # Use host audio
      - /etc/localtime:/etc/localtime:ro    # Use host timezone

volumes:
  tele-conf:                                # Persistent storage
  asterisk-conf:
     external: true
```

Aqui adicionaremos o armazenamento recém criado alterando as configurações do arquivo original para replicarem o exemplo abaixo. Apenas as áreas indicadas pelas setas verdes devem ser alteradas:

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/7a9d22e8-529e-4a31-b36f-67fb420beeaa)

Por fim, executamos o nosso container utilizando o comando:

```bash
$ make up
```

O Docker-compose fará o provisionamento do container com o Asterisk e mapeará as pastas de configuração no volume /data/asterisk-conf.

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/134ba4b8-abef-4d88-8086-c7109de0a2b2)

Com o contêiner iniciado poderemos acessar a cli do Asterisk através do comando “docker-compose exec tele asterisk -R” quer permite definir o modo de operação do Asterisk alterando as opções nativas (-r -R -rvvvv, etc) ou mesmo apenas o comando “make cli” que iniciará o cli com as opções -rvvvddd.

![image](https://github.com/zeraimundo/asterisk_docker/assets/82219488/c8a49a5b-6a86-42e7-ae71-d3f306770dba)

Este tutorial foi criado usando como fonte o site: https://computingforgeeks.com/how-to-run-asterisk-in-docker-container/
