# Projeto Prático AWS + Docker | #PB - NOV 2024 | DevSecOps

## Documentação do Projeto

Este projeto tem como objetivo configurar uma arquitetura de aplicação WordPress na AWS, containerizada (usando Docker), utilizando múltiplas zonas de disponibilidade, balanceamento de carga e dimensionamento automático para garantir alta disponibilidade e desempenho.

---

## Requisitos do projeto
- Instalação e configuração do DOCKER ou CONTAINER no host EC2.
- Ponto adicional para o trabalho: Utilizar a instalação via script de Start Instance (user_data.sh).
- Efetuar deploy de uma aplicação WordPress com container de aplicação RDS database MySQL.
- Configuração da utilização do serviço EFS AWS para estáticos do container de aplicação WordPress.
- Configuração do serviço de Load Balancer AWS para a aplicação WordPress.

## Pontos de atenção:
- Não utilizar IP público para saída dos serviços WordPress (Evitar publicar o serviço WordPress via IP público).
- Sugestão para o tráfego: Internet sair pelo LB (Load Balancer Classic).
- Pastas públicas e estáticos do WordPress sugestão de utilizar o EFS (Elastic File System).
- Fica a critério de cada integrante usar Dockerfile ou Docker Compose.
- Necessário demonstrar a aplicação WordPress funcionando (tela de login).
- Aplicação WordPress precisa estar rodando na porta 80 ou 8080.
- Utilizar repositório git para versionamento.
- Criar documentação.

## Ilustração
<img src=ilustracao-projeto.png>

---

## Execução prática

### Configuração da Rede:
- Acesse o console AWS e entre no serviço **VPC**.
- No menu lateral esquerdo, na seção de **Nuvem privada virtual** selecione **Suas VPCs**.
- Dentro de **Suas VPCs** clique no botão **Criar VPC**.
- Altere as seguintes configurações:
    - Em **Recursos a serem criados** selecione **VPC e muito mais**.
    - Em **Geração automática da etiqueta de nome** coloque o nome "VPCdocker".
    - Em **Número de zonas de disponibilidade (AZs)** selecione **2**.
    - Em **Gateways NAT** selecione **Em 1 AZ**.
    - Em **Endpoints da VPC** selecione **Nenhuma**.
- Clique em **Criar VPC**.

---

### Configuração dos Grupos de Segurança:
- Acesse o console AWS e entre no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Rede e segurança**, selecione **Security Groups**.
- Dentro de **Security Groups**, clique no botão **Criar grupo de segurança**.
- Crie e configure os seguintes grupos de segurança usando a VPC criada anteriormente:

    - #### Load Balancer - Regras de entrada
        | Tipo | Protocolo | Intervalo de portas |   Origem  |
        |:----:|:--------:|:-------------------:|:---------:|
        | HTTP | TCP      |          80         | 0.0.0.0/0 |

    - #### EC2 Web Server - Regras de entrada
        | Tipo | Protocolo | Intervalo de portas |       Origem       |
        |:----:|:--------:|:----------:|:------------------:|
        |  SSH |    TCP   |     22     |      EC2 ICE       |
        | HTTP |    TCP   |     80     |    Load Balancer   |

    - #### EC2 ICE - Regras de saída
        | Tipo | Protocolo | Intervalo de portas |       Origem       |
        |:----:|:--------:|:----------:|:------------------:|
        |  SSH |    TCP   |     22     |    EC2 Web Server  |

    - #### RDS - Regras de entrada
        |     Tipo     | Protocolo | Intervalo de portas |        Origem       |
        |:------------:|:--------:|:----------:|:-------------------:|
        | MYSQL/Aurora |    TCP   |    3306    |    EC2 Web Server   |

    - #### EFS - Regras de entrada
        | Tipo | Protocolo | Intervalo de portas |        Origem       |
        |:----:|:--------:|:----------:|:-------------------:|
        | NFS  | TCP      | 2049       |    EC2 Web Server   |

---

### Criando Elastic File System:
- Acesse o console AWS e entre no serviço de **EFS**.
- Na tela do **Elastic File System** clique no botão **Criar sistema de arquivos**.
- Depois clique no botão **Personalizar**.
- Execute a seguinte configuração: 

    - #### Etapa 1 - Configurações do sistema de arquivos:
        - No campo **Nome** digite "efs-docker".
        - Clique em **Próximo**.

    - #### Etapa 2 - Acesso à rede:
        - No campo **Virtual Private Cloud (VPC)** selecione a VPC que foi criada anteriormente.
        - No campo **ID da sub-rede** selecione as sub-redes privadas de cada AZ.
        - No campo **Grupos de segurança** selecione o grupo "EFS" que foi criado anteriormente.
        - Clique em **Próximo**.

    - #### Etapa 3 - Política do sistema de arquivos - opcional:
        - Clique em **Próximo**.
        
    - #### Etapa 4 - Revisar e criar:
        - Revise e clique em **Criar** para finalizar.

---

### Criando Relational Database Service:
- Acesse o console AWS e entre no serviço de **RDS**.
- Na tela **Painel** clique no botão **Criar banco de dados**.
- Execute a seguinte configuração:
    - Na seção **Opções do mecanismo** selecione **MySQL**.
    - Na seção **Modelos** selecione **Nível gratuito**.
    - Na seção **Configurações de credenciais** adicione uma *Senha principal" e confirme.
    - Na seção **Configuração da instância**, no campo selecione o tipo "db.t3.micro".
    - Na seção **Conectividade**, no campo **Nuvem privada virtual (VPC)** selecione a VPC criada anteriormente.
    - No campo **Grupos de segurança da VPC existentes** selecione o grupo "RDS" que foi criado anteriormente.
    - Na seção **Configuração adicional**, no campo **Nome do banco de dados inicial** coloque o nome "DBdocker".
- Revise e clique em **Criar banco de dados** para finalizar.

---

### Criando Load Balancer:
- Acesse o console AWS e entre no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Balanceamento de carga** selecione **Load balancers**.
- Dentro de **Load balancers** clique no botão **Criar load balancer**.
- Em **Tipos de load balancer** clique em **Classic Load Balancer** e depois em **Criar**.
- No campo **Nome do load balancer** digite "clb-wordpress".
- Na seção **Mapeamento de rede**, no campo **VPC** selecione a VPC criada anteriormente.
- No campo **Mapeamentos** selecione as duas AZ's e suas respectivas sub-redes públicas.
- No campo de **Grupos de segurança** selecione o grupo "Load Balancer" que foi criado anteriormente.
- Na seção **Verificações de integridade**, no campo **Caminho de ping** modifique o caminho para "/wp-admin/install.php".
- Clique em **Criar load balancer** para finalizar.

---

### Gerando Par de Chaves:
- Acesse o console AWS e entre no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Rede e segurança** selecione **Pares de chaves**.
- Dentro de **Pares de chaves** clique no botão **Criar par de chaves**.
- No campo **Nome** digite "chaveSSH". 
- No campo **Tipo de par de chaves** selecione **RSA**.
- No campo **Formato de arquivo de chave privada** selecione **.pem**.
- Clique no botão **Criar par de chaves**.
- Salve o arquivo .pem.

---

### Criando Modelo de Execução:
- Acesse o console AWS e entre no serviço **EC2**.
- No menu lateral esquerdo, na seção **Instâncias** selecione **Modelos de execução**.
- Dentro de **Modelos de execução** clique no botão **Criar modelo de execução**.
- No campo **Nome do modelo de execução** digite "me-wordpress".
- No campo **Descrição da versão do modelo** digite "wordpress-docker".
- Em **Imagens de aplicação e de sistema operacional** clique em **Início rápido**, depois clique em **Amazon Linux** e selecione a Amazon Linux 2023 AMI.
- No campo **Tipo de instância** selecione o tipo "t2.micro".
- No campo **Nome do par de chaves** selecione o par de chaves criado anteriormente.
- Em **Configurações de rede**, no campo **Grupos de segurança** selecione o grupo "EC2 Web Server" que foi criado anteriormente.
- Em **Tags de recurso** clique em **Adicionar nova tag** e adicione as tags de **Chave** "Name", "CostCenter" e "Project" para os **Tipos de recurso** "Instâncias" e "Volumes".
- Em **Detalhes avançados**, no campo **Dados do usuário** adicione o script abaixo:
```
#!/bin/bash

sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user

sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

sudo yum install -y amazon-efs-utils
sudo systemctl start amazon-efs-utils
sudo systemctl enable amazon-efs-utils

sudo mkdir /mnt/efs

sudo echo "fs-xxxxxxxxxxxx.efs.us-east-1.amazonaws.com:/ /mnt/efs nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0" >> /etc/fstab
sudo mount -a

sudo mkdir /mnt/efs/wordpress


sudo cat <<EOF > /mnt/efs/docker-compose.yaml
version: '3.8'
services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - 80:80
    environment:
      TZ: America/Sao_Paulo
      WORDPRESS_DB_HOST: endpoint_do_database
      WORDPRESS_DB_NAME: nome_do_database
      WORDPRESS_DB_USER: usuario_principal
      WORDPRESS_DB_PASSWORD: senha_usuario_principal
    volumes:
      - /mnt/efs/wordpress:/var/www/html
EOF

sudo yum install libxcrypt-compat -y
docker-compose -f /mnt/efs/docker-compose.yaml up -d
```
- Clique em **Criar modelo de execução** para finalizar.

---

### Criando Auto Scaling Group:
- Acesse o console AWS e entre no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Auto Scaling** selecione **Grupos Auto Scaling**.
- Dentro de **Grupos Auto Scaling** clique no botão **Criar grupo do Auto Scaling**.
- Execute a seguinte configuração:

    - #### Etapa 1 - Escolher o modelo de execução:
        - No campo **Nome do grupo do Auto Scaling** digite "asg-wordpress".
        - Na seção **Modelo de execução** selecione o modelo criado anteriormente.
        - Clique em **Próximo**.
  
    - #### Etapa 2 - Escolher as opções de execução de instância:
        - Na seção **Rede**, no campo **VPC** selecione a VPC criada anteriormente.
        - No campo **Zonas de disponibilidade e sub-redes** selecione as duas sub-redes privadas criadas previamente.
        - Clique em **Próximo**.
  
    - #### Etapa 3 - Integrar com outros serviços:
        - Na seção **Balanceamento de carga** selecione **Anexar a um balanceador de carga existente**.
        - Na seção **Anexar a um balanceador de carga existente** clique em **Escolha entre Classic Load Balancers** e selecione o load balancer criado anteriormente.
        - Na seção **Verificações de integridade** marque a opção **Ativar verificações de integridade do Elastic Load Balancing**.
        - Clique em **Próximo**.
  
    - #### Etapa 4 - Configurar tamanho do grupo e ajuste de escala:
        - No campo **Capacidade desejada** digite "2".
        - Em **Escalabilidade**, no campo **Capacidade mínima desejada** digite "2".
        - No campo **Capacidade máxima desejada** digite "4".
        - Em **Escalabilidade automática** selecione a opção **Política de escalabilidade de rastreamento de destino**
        - No campo **Tipo de métrica** deixe selecionado **Média de utilização da CPU**.
        - No campo **Valor de objetivo** digite "75".
        - Clique em **Próximo**.
  
    - #### Etapas 5, 6 e 7:
        - Clique em **Próximo**.
        - Clique em **Próximo**.
        - Revise e clique em **Criar grupo do Auto Scaling** para finalizar.

---

### Configuração EC2 Instance Connect Endpoint:
- Acesse o console AWS e entre no serviço **VPC**.
- No menu lateral esquerdo, na seção de **PrivateLink and Lattice** selecione **Endpoints**.
- Dentro de **Endpoints** clique no botão **Criar endpoint**.
- Altere as seguintes configurações:
    - Em **Etiqueta de nome** coloque o nome "endpoint-wordpress".
    - Em **Tipo** selecione **Endpoint do EC2 Instance Connect**.
    - Em **VPC** selecione a VPC criada anteriormente.
    - Em **Grupos de segurança** selecione o grupo "EC2 ICE" que foi criado anteriormente.
    - Em **Sub-rede** selecione uma sub-rede privada que foi criada anteriormente.
- Clique em **Criar endpoint**.

---

### Acesso e Teste:
- Acesse o **Nome do DNS** do **Load Balancer** através do navegador.
- Na tela de instalação do **WordPress** escolha o idioma e cliquei em **Continuar**.
- Na tela seguinte preencha os dados para criação de um usuário.
- Clique em **Instalar WordPress** para finalizar.
 
