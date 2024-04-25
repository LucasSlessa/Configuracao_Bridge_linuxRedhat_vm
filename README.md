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
  
5. Desabilite o selinux(nao necessario)

## Atualização e Instalação de Pacotes

1. Atualize o sistema:

   ```bash
   dnf update -y
   dnf install epel-release -y 
   dnf install bind bind-utils vim-enhanced bash-completion bmon vim htop iftop net-tools git -y
   reboot 
   ```





