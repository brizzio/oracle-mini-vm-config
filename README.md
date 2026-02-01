# oracle-mini-vm-config
Micro (AMD): 1 CPU e 1 GB de RAM Para uma instância de 1 GB de RAM

1. Documento Explicativo das Configurações
Aqui está o que cada comando faz no seu servidor:

A. Memória e Sistema (Swap)
Como sua instância tem apenas 1 GB de RAM, criamos um "fôlego" extra no disco.

fallocate -l 2G /swapfile: Cria um arquivo de 2 GB.

chmod 600 /swapfile: Protege o arquivo para que apenas o sistema o acesse.

mkswap e swapon: Transformam o arquivo em memória virtual e o ativam.

vm.swappiness=10: Diz ao Linux para usar o disco apenas em última instância, priorizando a RAM real.

B. Instalação do Docker
Configuramos o repositório oficial para garantir que você receba atualizações de segurança.

mkdir -p /etc/apt/keyrings: Cria a pasta para as chaves de segurança do Docker.

curl ... | gpg: Baixa a chave de autenticação do Docker.

apt install docker-ce...: Instala o motor do Docker e o plugin do Compose.

usermod -aG docker $USER: Permite que você use o comando docker sem precisar digitar sudo toda hora.

C. Firewall (iptables)
O Ubuntu na Oracle Cloud vem com regras rígidas. Além do painel da Oracle, o Linux precisa permitir o tráfego interno.

iptables -I INPUT 6 ...: Insere uma regra de "Permitir" na lista de conexões.

netfilter-persistent save: Grava essa regra para que ela não seja apagada ao reiniciar o servidor.

2. Bash Script de Configuração Completa
Copie o código abaixo e salve-o como um arquivo chamado setup_server.sh. Nota: Este script resume tudo o que fizemos manualmente até agora.

``` bash

#!/bin/bash

# --- 1. ATUALIZAÇÃO DO SISTEMA ---
sudo apt update && sudo apt upgrade -y

# --- 2. CONFIGURAÇÃO DE SWAP (2GB) ---
if [ ! -f /swapfile ]; then
    sudo fallocate -l 2G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
    echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p
fi

# --- 3. INSTALAÇÃO DO DOCKER ---
sudo apt install ca-certificates curl gnupg -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo usermod -aG docker $USER

# --- 4. CONFIGURAÇÃO DO FIREWALL (PORTAS PORTAINER E STRAPI) ---
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 9443 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 1337 -j ACCEPT
sudo apt install iptables-persistent -y
sudo netfilter-persistent save

# --- 5. INSTALAÇÃO DO PORTAINER ---
sudo docker volume create portainer_data
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

echo "Configuração finalizada! Acesse https://seu-ip:9443"

```

3. Como usar o script?
Se você quiser rodar esse script em uma nova instância no futuro, o processo é este:

Crie o arquivo no servidor: 

``` bash
nano setup_server.sh
```

Cole o código acima dentro do editor. (Para sair e salvar no nano: Ctrl+O, Enter e Ctrl+X).

Dê permissão de execução ao arquivo:

``` bash
chmod +x setup_server.sh
```

Execute o script:

``` bash
./setup_server.sh
```
