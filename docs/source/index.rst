Tutorial para subir Aplicação Web CRUD em ambiente PROD.

Passo 1) Crie a máquina virtual PROD:
  multipass launch -n serverprod lts
  
  - Acesse a máquina virtual Prod:
  multipass shell serverprod

Passo 2) Banco de Dados:
 - Crie o container:
   docker container run -d --name dbassist ubuntu:jammy /bin/bash -c "sleep infinity"
   
 - Defina a localização geográfica para que a data e hora fiquem corretas:
   apt-get update
   apt-get install tzdata

   Selecione Time Zone "135" e tecle ENTER
   obs: você pode verificar se a data e hora estão corretas digitando o comando "date".
   
 - Instale o Mysql no container:
   apt-get install mysql-server
  
 - Inicialize o Mysql:
   /etc/init.d/mysql start
  
   obs: você pode verificar se ele realmente inicializou nos processos que estão rodando no container, via comando "ps aux".
   
 - Abra a porta do mysql:
   nano /etc/mysql/mysql.conf.d/mysqld.cnf
   
   Altere o bind-address para "0.0.0.0".
  
 - Acesse o Mysql:
   mysql -u root -p
  
   obs: em seguida digite a senha da sua máquina física.
  
 - Crie seu Banco de Dados no Mysql:
   create database assist;
   use assist;
  
 - Crie seu Usuário no Banco de Dados e conceda permissão para ele:
   create user 'assist'@'%' identifiedy by 'assist123';
   grant all privileges on assist.* to 'assist'@'%';
  
 - Crie a tabela customers no Banco de Dados:
   CREATE TABLE `customers` (
     `customer_id` INT(11) NOT NULL AUTO_INCREMENT,
     `firstname` VARCHAR(255) NULL DEFAULT NULL,
     `lastname` VARCHAR(255) NULL DEFAULT NULL,
     `email` VARCHAR(255) NULL DEFAULT NULL,
     `created` DATETIME NULL DEFAULT NULL,
     PRIMARY KEY (`customer_id`)
   ) COLLATE = 'latin1_swedish_ci' ENGINE = InnoDB AUTO_INCREMENT = 1;
   
   obs: você pode verificar se a mesma foi criada utilizando o comando "show tables;"
  
Passo 3) Servidor Web:
 - Crie o container:
   docker container run -d --name webassist -p 80:80 ubuntu:jammy /bin/bash -c "sleep infinity"
   
 - Defina a localização geográfica para que a data e hora fiquem corretas:
   apt-get update
   apt-get install tzdata
   
   Selecione Time Zone "135" e tecle ENTER
   obs: você pode verificar se a data e hora estão corretas digitando o comando "date".
  
 - Instale o Apache2 no container:
   apt-get install apache2
   
 - Inicialize o Apache2:
   /etc/init.d/apache2 start
   
   obs: você pode verificar se ele realmente inicializou nos processos que estão rodando no container, via comando "ps aux".
   
 - Instale a libraria de conexão entre o Apache2 e o Php:
   apt-get install libapache2-mod-php
 
 - Clone os arquivos no diretório Var:
   cd /var/www/html/
   git clone git@github.com:josejunior2023/crudjjtec.git
   
Passo 4) Crie a máquina virtual DNS:
  multipass launch -n dns lts
  
  - Acesse a máquina virtual DNS:
    multipass shell dns
  
  - Instale o Servidor de DNS bind9:
    sudo apt-get update
    sudo apt-get install bind9
  
  - Altere o Servidor DNS bind9:
    sudo nano /etc/bind/named.conf.options
  
    altere a linha "listen-on-v6 {any};" para "listen-on-v6{};" para que ele não escute ipv6.
    altere a linha "dnssec-validation auto;" para "dnssec-validaton no;" para retirar o dns seguro.
    insira a linha "allow-query { [SUA_REDE]/24; 127.0.0.0/8; };" após a linha "dnssec-validation no;".
  
  - Reinicie o Servidor de DNS bind9:
    sudo systemctl restart bind9
    
    obs: você pode verificar se o serviço está rodando utilizando o comando "sudo systemctl status bind9".
   
  - Crie o domínio www.jjtec.com.br:
    sudo nano /etc/bind/named.conf.local
    
    insira as linhas:
     zone "jjtec.com.br" IN {
     	      type master;
     	      file "db.jjtec.com.br";
     };	      
   
  - Crie o arquivo db.jjte.com.br:
    sudo nano /var/cache/bind/db.jjte.com.br
    
    insira as linhas:
     $ORIGIN jjtec.com.br.
     $TTL 300;
     @ IN SOA dns josejunior567.hotmail.com (1 30 30 30 30);
     @ IN NS dns
     dns IN A [IP_SUA_MAQ_VIRTUAL_DNS]
     web IN CNAME @
     @ IN A [IP_SUA_MAQ_VIRTUAL_PROD]
     www IN CNAME @
     
  - Reinicie o Servidor de DNS bind9:
    sudo systemctl restart bind9
    
    obs: você pode verificar se o serviço está rodando utilizando o comando "sudo systemctl status bind9".
    
  - Altere o arquivo head em sua máquina física:
    sudo nano /etc/resolvconf/resolv.conf.d/head
    
    insira a linha "nameserver [IP_SUA_MAQ_VIRTUAL_DNS]"
    
  - Atualize o resolvconf:
    sudo resolvconf -u
