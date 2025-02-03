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
