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
- Acessei o console AWS e entrei no serviço **VPC**.
- No menu lateral esquerdo, na seção de **Virtual private cloud** selecionei **Your VPCs**.
- Dentro de **Your VPCs** cliquei no botão **Create VPC**.
- Alterei as seguintes configurações:
    - Em **Resources to create** selecionei **VPC and more**.
    - Em **Name tag auto-generation** coloquei o nome "docker-vpc".
    - Em **Number of Availability Zones (AZs)** selecionei **2**.
    - Em **NAT gateways** selecionei **In 1 AZ**.
    - Em **VPC endpoints** selecionei **None**.
- Cliquei em **Create VPC**.

---

### Configuração dos Grupos de Segurança:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Network & Security**, selecionei **Security Groups**.
- Dentro de **Security Groups**, cliquei no botão **Create security group**.
- Criei e configurei os seguintes security groups usando a VPC criada anteriormente:

    - #### Load Balancer - Inbound rules
        | Type | Protocol | Port Range |   Source  |
        |:----:|:--------:|:----------:|:---------:|
        | HTTP | TCP      | 80         | 0.0.0.0/0 |

    - #### EC2 Web Server - Inbound rules
        | Type | Protocol | Port Range |       Source       |
        |:----:|:--------:|:----------:|:------------------:|
        |  SSH |    TCP   |     22     |      EC2 ICE       |
        | HTTP |    TCP   |     80     |    Load Balancer   |

    - #### EC2 ICE - Outbound rules
        | Type | Protocol | Port Range |       Source       |
        |:----:|:--------:|:----------:|:------------------:|
        |  SSH |    TCP   |     22     |    EC2 Web Server  |

    - #### RDS - Inbound rules
        |     Type     | Protocol | Port Range |        Source       |
        |:------------:|:--------:|:----------:|:-------------------:|
        | MYSQL/Aurora |    TCP   |    3306    |    EC2 Web Server   |

    - #### EFS - Inbound rules
        | Type | Protocol | Port Range |        Source       |
        |:----:|:--------:|:----------:|:-------------------:|
        | NFS  | TCP      | 2049       |    EC2 Web Server   |

---

### Criando Elastic File System:
- Acessei o console AWS e entrei no serviço de **EFS**.
- Na tela do **Elastic File System** cliquei no botão **Create file system**.
- Depois cliquei no botão **Customize**.
- Executei a seguinte configuração: 

    - #### Step 1 - File system settings:
        - No campo **Name** digitei "efs-docker".
        - Cliquei em **Next**.

    - #### Step 2 - Network access:
        - No campo **Virtual Private Cloud (VPC)** selecionei a VPC que foi criada anteriormente.
        - No campo **Subnet ID** selecionei as subnets privadas de cada AZ.
        - No campo **Security groups** selecionei o grupo "EFS" que foi criado anteriormente.
        - Cliquei em **Next**.

    - #### Step 3 - optional - File system policy:
        - Cliquei em **Next**.
        
    - #### Step 4 - Review and create:
        - Revisei e cliquei em **Create** para finalizar.

---

### Criando Relational Database Service:
- Acessei o console AWS e entrei no serviço de **RDS**.
- Na tela **Dashboard** cliquei no botão **Create database**.
- Executei a seguinte configuração:
    - Na seção **Engine options** selecionei **MySQL**.
    - Na seção **Templates** selecionei **Free tier**.
    - Na seção **Credentials Settings** adicionei uma *Master password* e confirmei.
    - Na seção **Conectivity**, no campo **Virtual private cloud** selecionei a VPC criada anteriormente.
    - No campo **Existing VPC security groups** selecionei o grupo "RDS" que foi criado anteriormente.
    - Na seção **Additional configuration**, no campo **Initial database name** coloquei o nome "dockerdb".
- Revisei e cliquei em **Create database** para finalizar.

---

### Criando Load Balancer:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Load Balancing** selecionei **Load Balancers**.
- Dentro de **Load Balancers** cliquei no botão **Create load balancer**.
- Em **Load balancer types** cliquei em **Classic Load Balancer** e depois em **Create**.
- No campo **Load balancer name** digitei "ws-clb".
- Na seção **Network mapping**, no campo **VPC** selecionei a VPC criada anteriormente.
- No campo **Mappings** selecionei as duas AZ's e suas respectivas subnets públicas.
- No campo de **Security groups** selecionei o grupo "Load Balancer" que foi criado anteriormente.
- Na seção **Health checks**, no campo **Ping path** adicionei o caminho "/wp-admin/install.php".
- Cliquei em **Create load balancer** para finalizar.

---

### Gerando Par de Chaves:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Network & Security** selecionei **Key pairs**.
- Dentro de **Key pairs** cliquei no botão **Create key pair**.
- No campo **Name** digitei "MinhaChaveSSH". 
- No campo **Key pair type** selecionei **RSA**.
- No campo **Private key file format** selecionei **.pem**.
- Cliquei no botão **Create key pair**.
- Salvei o arquivo .pem.

---

### Criando Modelo de Execução:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção **Instances** selecionei **Launch Templates**.
- Dentro de **Launch Templates** cliquei no botão **Create launch template**.
- No campo **Launch template name** digitei "ws-lt".
- No campo **Template version description** digitei "docker-wordpress".
- Em **Application and OS Images** cliquei em **Quick Start**, depois cliquei em **Amazon Linux** e selecionei a Amazon Linux 2023 AMI.
- Na seção **Instance type** selecionei o tipo t3.small.
- No campo **Key pair name** selecionei a key pair criada anteriormente.
- Em **Network settings**, no campo **Security groups** selecionei o grupo "EC2 Web Server" que foi criado anteriormente.
- Em **Resource tags** cliquei em **Add new tag** e adicionei as tags de **Key** "Name", "CostCenter" e "Project" para os **Resource types** Instances e Volumes.
- Em **Advanced details**, no campo **User data** adicionei o script abaixo:
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
- Cliquei em **Create launch template** para finalizar.

---

### Criando Auto Scaling Group:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Auto Scaling** selecionei **Auto Scaling Groups**.
- Dentro de **Auto Scaling groups** cliquei no botão **Create Auto Scaling group**.
- Executei a seguinte configuração:
    - #### Step 1 - Choose launch template:
        - No campo **Auto Scaling group name** digitei "ws-asg".
        - Na seção **Launch template** selecionei o template criado anteriormente.
        - Cliquei em **Next**.
    - #### Step 2 - Choose instance launch options:
        - Na seção **Network**, no campo **VPC** selecionei a VPC criada anteriormente.
        - No campo **Availability Zones and subnets** selecionei as duas subnets privadas criadas previamente.
        - Cliquei em **Next**.
    - #### Step 3 - Configure advanced options:
        - Na seção **Load balancing** selecionei **Attach to an existing load balancer**.
        - Na seção **Attach to an existing load balancer** cliquei em **Choose from Classic Load Balancers** e selecionei o load balancer criado anteriormente.
        - Na seção **Health checks** marquei a opção **Turn on Elastic Load Balancing health checks**.
        - Cliquei em **Next**.
    - #### Step 4 - Configure group size and scaling:
        - No campo **Desired capacity** digitei "2".
        - Em **Scaling**, no campo **Min desired capacity** digitei "2".
        - No campo **Max desired capacity** digitei "4".
        - Em **Automatic scaling** selecionei a opção **Target tracking scaling policy**
        - No campo **Metric type** deixei selecionado **Average CPU utilization**.
        - No campo **Target value** digitei "75".
        - Cliquei em **Next**.
    - #### Steps 5, 6 e 7:
        - Cliquei em **Next**.
        - Cliquei em **Next**.
        - Revisei e cliquei em **Create Auto Scaling group** para finalizar.

---

### Configuração EC2 Instance Connect Endpoint:
- Acessei o console AWS e entrei no serviço **VPC**.
- No menu lateral esquerdo, na seção de **Virtual private cloud** selecionei **Endpoints**.
- Dentro de **Endpoints** cliquei no botão **Create endpoint**.
- Alterei as seguintes configurações:
    - Em **Name tag** coloquei o nome "ws-ep".
    - Em **Service category** selecionei **EC2 Instance Connect Endpoint**.
    - Em **VPC** selecionei a VPC criada anteriormente.
    - Em **Security groups** selecionei o grupo "EC2 ICE" que foi criado anteriormente.
    - Em **Subnet** selecionei uma subnet privada que foi criada anteriormente.
- Cliquei em **Create endpoint**.

---

### Acesso e Teste:
- Acessei o **DNS name** do **Load Balancer** através do navegador.
- Na tela de instalação do **WordPress** mantive o idioma padrão e cliquei em **Continue**.
- Na tela seguinte preenchi os dados para criação de um usuário.
- Cliquei em **Install WordPress** para finalizar.
 
