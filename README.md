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

### Configuração Network:
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

### Configuração de Security Groups:
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

### Gerando Key pair:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Network & Security** selecionei **Key pairs**.
- Dentro de **Key pairs** cliquei no botão **Create key pair**.
- No campo **Name** digitei "MinhaChaveSSH". 
- No campo **Key pair type** selecionei **RSA**.
- No campo **Private key file format** selecionei **.pem**.
- Cliquei no botão **Create key pair**.
- Salvei o arquivo .pem.

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

