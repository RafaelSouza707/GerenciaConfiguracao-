# Instalando e configurando acesso remoto e firewall em um sistema linux

### Tenha o docker instalado
* Verifique se está instalado

      docker --version

### Criando um container Ubuntu 24.04
* Baixe a imagem oficial

      docker pull ubuntu:24.04

### Crie e execute um container

        docker run -d \
        --name ubuntu2404 \
        --cap-add=NET_ADMIN \
        --cap-add=NET_RAW \
        -p 2222:22 \
        ubuntu:24.04 \
        bash -c "tail -f /dev/null"

#### ou
        docker run -d \
        --name ubuntu2404 \
        --privileged \
        -p 2222:22 \
        ubuntu:24.04 \
        bash -c "tail -f /dev/null"
##### Detalhe:
    run -> Criar e iniciar um container novo a partir de uma imagem
    
    cap-add=NET_ADMIN -> permite administrar rede dentro do container (iptables, ufw, rotas, interfaces).
    cap-add=NET_RAW -> permite operações de rede em nível mais baixo (raw sockets), útil para algumas funções de rede e ferramentas que usam raw sockets (ex: ping).
    
    privileged → concede permissões quase totais ao container, permitindo acesso ampliado a recursos (não é recomendado em ambientes reais, pois reduz bastante a segurança.)

    -d -> Detached mode (modo desacoplado), Isso quer dizer que o container roda em segundo plano (background), e o terminal não fica preso nele.
    
    --name ubuntu2404 -> Define um nome fixo para o container
    
    -p 2222:22 -> Faz mapeamento de portas, onde: -p PORTA_HOST:PORTA_CONTAINER
    bash -> É o programa principal que será executado dentro do container
    
    -c -> Execute o comando que vem logo em seguida como uma string.
    "tail -f /dev/null" -> 
        Executado pelo "-c"; 
        tail -> normalmente mostra o final de um arquivo; 
        -f -> significa "follow", acompanhar continuamente.

### Entrar no prompt do container

    docker exec -it ubuntu2404  /bin/bash

### Atualizar pacotes do sistema
* Dentro do container:

      apt update && apt upgrade -y

### Instlando o OpenSSH Server
* Instale o servidor SSH

      apt install openssh-server -y

* Verifique se o SSH foi instalado corretamente

      service ssh status
      ssh -V

### Criando Usuário para Acesso SSH
* Crie um usuário novo (exemplo: admin):

      adduser admin
    
* Adicione o usuário ao grupo sudo

      usermod -aG sudo admin

* Verifique

      groups admin

### Configurando o SSH
* Abra o arquivo de configuração:

      nano /etc/ssh/sshd_config

* Permitir login por senha (caso esteja desabilitado)

      PasswordAuthentication yes

* Desabilitar login direto como root (recomendado)

      PermitRootLogin no

* Garantir que SSH esteja na porta padrão 22

      Port 22

* Salve e saia

### Iniciando o Serviço SSH

    service ssh start

* Verifique o status:

      service ssh status

### Testando Acesso SSH no Host
* No computador host (fora do container), execute:

      ssh admin@localhost -p 2222

* Digite a senha criada para o usuário.

### Instalando o Firewall UFW
* Dentro do container:

      apt install ufw -y

* Verifique a versão:

      ufw --version

### Configurando Regras Básicas do Firewall
* Bloquear tudo por padrão

      ufw default deny incoming

* Permitir tudo por padrão

      ufw default allow outgoing

### Liberar porta SSH

    ufw allow 22/tcp

* Verifique regras adicionadas:

      ufw status verbose

### Ativando o Firewall
* Ative o UFW:

      ufw enable

* Verifique se está ativo

      ufw status

### Verificar portas abertas
* Dentro do container instale net-tools:

      apt install net-tools -y

* Verifique portas em uso:

      netstat -tulnp

### Testar SSH novamente

    ssh admin@localhost -p 2222

* Se conectar normalmente, significa que:

* SSH está funcionando
* Firewall está permitindo porta 22

### Boas Práticas Recomendadas
* No host, gere chave:

      ssh-keygen

* Copie a chave para o container:
    
      ssh-copy-id -p 2222 admin@localhost

* Agora edite novamente, com nano ou qualquer outro editor de texto de sua preferencia:
    
      nano /etc/ssh/sshd_config

* Ajuste o campo:
    
      PasswordAuthentication no

* Reinicie o serviço:

      service ssh restart

### Encerrando e Reiniciando o Container
* Para sair do container sem encerrá-lo:

      exit
      
* Para parar:
  
      docker stop ubuntu2404
      
* Para iniciar novamente:

      docker start ubuntu2404
