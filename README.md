# Configuracao_Bridge_linuxRedhat_vm




# Configuração de Rede

## configuração da primeira maquina
## Passo 1: Configuração da Placa em Modo Bridge

1. Configure a placa 1 em modo bridge (selecionargb ou wifi).
2. Configure a placa 2 o modo de rede exclusiva do hospedeiro para GB Ethernet.
3. Nomeie a máquina como GW-NAT(minusculo).

## Passo 2: Configuração de Senha e Rede

1. Configure uma senha.
2. Nas configurações de rede, defina o hostname como gw-nat.
3. Escolha uma faixa de IP: 192.168.56.11.
4. Na configuração da segunda placa, altere o IPv4 para manual e defina a faixa de rede acima, com uma máscara de 24.
5. Marque a checkbox em "require IPv4".
6. Desative o IPv6.
7. nao esqueça de dar allow root e nao definir nome da maquina

## Configuração do Roteador

1. Desabilite o SELinux:

   ```bash
   sudo vi /etc/selinux/config
   ```

   Defina `SELINUX=disabled`.

2. Para fazer SSH no CMD, use `ssh root@ip_que_estamos`.

## Atualização e Instalação de Pacotes

1. Atualize o sistema:

   ```bash
   dnf update -y
   reboot
   ```

2. Instale o repositório EPEL:

   ```bash
   dnf install epel-release -y
   dnf install bmon vim htop iftop net-tools bash-completion bind-utils -y
   ```

## Regras do Firewall

1. Adicione interfaces ao firewall:

   ```bash
   firewall-cmd --zone=external --add-interface=enp0s3 --permanent
   firewall-cmd --zone=internal --add-interface=enp0s8 --permanent
   ```

2. Defina a zona padrão para external:

   ```bash
   firewall-cmd --set-default-zone=external 
   ```

3. Recarregue o firewall:

   ```bash
   firewall-cmd --reload
   ```

4. Verifique a zona padrão:

   ```bash
   firewall-cmd --get-default-zone
   ```

5. Crie uma nova política:

   ```bash
   firewall-cmd --new-policy internal-external --permanent
   firewall-cmd --reload
   ```

6. Adicione zonas de ingresso e egresso à política:

   ```bash
   firewall-cmd --policy internal-external --add-ingress-zone=internal --permanent
   firewall-cmd --policy internal-external --add-egress-zone=external --permanent
   ```

7. Defina o alvo como ACCEPT:

   ```bash
   firewall-cmd --policy internal-external --set-target=ACCEPT --permanent
   ```

8. Recarregue o firewall:

   ```bash
   firewall-cmd --reload
   ```

9. Verifique as informações da política:

   ```bash
   firewall-cmd --info-policy internal-external
   ```

    ```bash
   route
   ```
     ```bash
   bmon
      ```
    ```bash
   chronyc
   tracking
   ```

# Configuração da Segunda Máquina autoridade dns

1. Nome: namesrv01
2. Placa única em modo rede exclusiva do hospedeiro GB Ethernet.
3. Configuração de rede:

     nmtui (ou configura na tela antes da instalação)

   
    nosso ip 192.168.56.12/24
   
   - Hostname: namesrv01.sd.aula
   - Gateway: 192.168.56.11(apontamento pra rede anterior)
   - DNS: 8.8.8.8
   - Search Domain: sd.aula
  
4. clique em allow require e dasative o ipv6.
  
5. Desabilite o selinux

## Atualização e Instalação de Pacotes

1. Atualize o sistema:

   ```bash
   dnf update -y
   dnf install epel-release -y 
   dnf install bind bind-utils vim-enhanced bash-completion bmon vim htop iftop net-tools git -y
   reboot 
   ```


2. criando a pasta dos logs

   ```bash

   mkdir /var/named/log/
   chown named:named /var/named/log/
   chmod 700 /var/named/log/

   ```

3. configuração do firewall

   ```bash
   firewall-cmd --remove-service=dhcpv6-client --remove-service=cockpit --permanent
   firewall-cmd --add-service=dns --permanent
   firewall-cmd --reload
   firewall-cmd --list-all
   ```

4. Verificação
   ```bash
   route
   ```

5. Configuração do dns

  ```bash
   vi /etc/named.conf
   ```

   ```vi
           
      //
      // named.conf
      //
      // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
      // server as a caching only nameserver (as a localhost DNS resolver only).
      //
      // See /usr/share/doc/bind*/sample/ for example named configuration files.
      //
      
      options {
      	listen-on port 53 { 127.0.0.1; 192.168.56.12; };
      	#listen-on-v6 port 53 { ::1; };
      	directory 	"/var/named";
      	dump-file 	"/var/named/data/cache_dump.db";
      	statistics-file "/var/named/data/named_stats.txt";
      	memstatistics-file "/var/named/data/named_mem_stats.txt";
      	secroots-file	"/var/named/data/named.secroots";
      	recursing-file	"/var/named/data/named.recursing";
      	allow-query     { internal-networks; dmz-networks; }; # { any; };
      	allow-recursion { internal-networks; dmz-networks; }; # { any; };
      	forwarders { 8.8.8.8; };
      
      	/* 
      	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
      	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
      	   recursion. 
      	 - If your recursive DNS server has a public IP address, you MUST enable access 
      	   control to limit queries to your legitimate users. Failing to do so will
      	   cause your server to become part of large scale DNS amplification 
      	   attacks. Implementing BCP38 within your network would greatly
      	   reduce such attack surface 
      	*/
      	recursion yes;
      
      	dnssec-validation no;
      
      	managed-keys-directory "/var/named/dynamic";
      	geoip-directory "/usr/share/GeoIP";
      
      	pid-file "/run/named/named.pid";
      	session-keyfile "/run/named/session.key";
      
      	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
      	include "/etc/crypto-policies/back-ends/bind.config";
      };
      
      acl internal-networks { 127.0.0.1; 192.168.56.0/24; };
      acl dmz-networks { 0.0.0.0; };
      
      logging {
              channel default_debug {
                      file "data/named.run";
                      severity dynamic;
              };
      	category notify { zone_transfer_log; };
      	category xfer-in { zone_transfer_log; };
      	category xfer-out { zone_transfer_log; };
      	channel zone_transfer_log {
      		file "/var/named/log/transfer.log" versions 10 size 300m;
      		print-time yes;
      		print-category yes;
      		print-severity yes;
      		severity info;
      	};
      };
      
      zone "." IN {
      	type hint;
      	file "named.ca";
      };
      
      
      zone "sd.aula" {
      	type master;
      	file "sd.aula.zone";
      	allow-query { any; };
      	dnssec-policy default;
      };
      
      zone "56.168.192.in-addr.arpa" {
      	type master;
      	file "56.168.192.in-addr.arpa.zone";
      	allow-query { any; };
      	dnssec-policy default;
      
      };
      
      
      include "/etc/named.rfc1912.zones";
      include "/etc/named.root.key";

   ```


6. Verificação
   ```bash
     named-checkconf
     ll
   ```



7. Criar as zonas
   ```bash
   cd /named
   touch sd.aula.zone
   touch 56.168.192.in-addr.arpa.zone
   ```

8. MUDAR DONOS
   ```bash
   chown named *.zone
   chmod 600 *.zone   
    ```

zona sd.aula
```vi

$ORIGIN .
$TTL 86400	; 1 dia
sd.aula.		IN SOA	namesvr01.sd.aula. eduschiavo.univap.br. (
				2024042501 ; serial
				10800	; refresh (3 horas)
				3600	; retry (1 hora)
				604800	; expire (1 semana)
				10	; minimun (1 dia)
				)
			NS	namesvr01.sd.aula.
$ORIGIN sd.aula.
$TTL 10		; 1 hora
gw-nat			A	192.168.56.11
namesvr01		A	192.168.56.12

```

zona reversa

```vi
$ORIGIN .
$TTL 86400	; 1 dia
56.168.192.in-addr.arpa.	IN SOA	namesvr01.sd.aula. eduschiavo.univap.br. (
				2024042501 ; serial
				10800	; refresh (3 horas)
				3600	; retry (1 hora)
				604800	; expire (1 semana)
				10	; minimun (1 dia)
				)
			NS	namesvr01.sd.aula.
$ORIGIN 56.168.192.in-addr.arpa.
11			PTR	gw-nat.sd.aula.
12			PTR	namesvr01.sd.aula.
```






11. start e habilitação/ verificação

    ```bash

    ```

    
    ```bash
    # check dig local
      dig @localhost namesvr01.sd.aula
      dig @localhost gw-nat.sd.aula
      dig @localhost uol.com.br
      dig @localhost gmail.com
      
      # check dig remoto
      ping 192.168.56.12
      dig @192.168.56.12 namesvr01.sd.aula
      dig @192.168.56.12 gw-nat.sd.aula
      dig @192.168.56.12 uol.com.br
      dig @192.168.56.12 gmail.com
    ```

    
   





