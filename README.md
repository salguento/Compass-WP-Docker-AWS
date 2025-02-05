# Compass-WP-Docker-AWS
Segunda atividade prática do programa de bolsas de DevSecOps da CompassUOL.


## Descrição do projeto 

Objetivo -> Criar um ambiente na AWS, para rodar um site Wordpress com Docker, em EC2, com conexão RDS e EFS, para base de dados e arquivos, em duas zonas de disponibilidade (Availability Zones), com Load Balancer e Auto Scaling Group.

## VPC - Virtual Private Compute

- Crie uma VPC com CIDR
- Crie Subnets:
	- 2 Subnets públicas (AZ1 e AZ2) - Para o Load Balancer
	- 2 Subnets privadas (AZ1 e AZ2) - Para EC2 e RDS
- Internet Gateway anexado a VPC
- NAT Gateway em cada subnet pública ( se acesso a internet for necessário)
- Route Tables
	- Rota das subnets públicas 0.0.00/0 -> Internet Gateway
	- Rota das subnets privadas 0.0.0.0/0 -> NAT Gateway

## Grupos de Segurança

- Crie grupos de segurança para EC2, RDS, EFS, e ALB

- Grupo de segurança para ALB
```
Name: LB-SG
Inbound:
  - Type: HTTP (80), Source: 0.0.0.0/0
  - Type: HTTPS (443), Source: 0.0.0.0/0
Outbound:
  - Type: All traffic, Destination: EC2-SG (or port 80)
```

- Grupo de segurança para EC2

```
Name: EC2-SG
Inbound:
  - Type: HTTP (80), Source: LB-SG
  - Type: SSH (22), Source: <your-IP> (optional)
Outbound:
  - Type: All traffic, Destination: 0.0.0.0/0
```

- Grupo de segurança para RDS

```
Name: RDS-SG
Inbound:
  - Type: MySQL/Aurora (3306), Source: EC2-SG
```

- Grupo de segurança para EFS

```
Name: EFS-SG
Inbound:
  - Type: NFS (2049), Source: EC2-SG
```

## EFS - Elastic File System

- Crie um sistema de arquivo EFS
- Anote o file system id do EFS

## RDS - MySQL

- Crie uma instância RDS MySQL com Multi AZ habilitado
- Anote as informações da base de dados:
	- Endpoint
	- Nome da base de dados
	- Usuário
	- Senha

## Systems Managers Parameter Store

- Adicione as variáveis de ambiente individualmente com os valores anotados do EFS e RDS
	- efs-id -> EFS file system ID
	- db-host -> Endpoint
	- db-name -> Nome
	- db-user -> User
	- db-password -> Password

## ALB - Automatic Load Balancer

- Crie o Load Balancer nas subnets públicas
- Adicione o grupo de segurança
- Configure o Target Group
	- Protocol : HTTP:80
	- Health Check : `/wp-admin/install.php` (HTTP 200)

## ASG - Auto Scaling Group

- Crie um template
	- Use uma imagem Amazon Linux 2
	- Grupo de segurança EC2
	- IAM role EC2RoleForSSM
	- Adicione o arquivo `user_data.sh` como user data
- Crie o ASG
	- Selecione as 2 subnets privadas (AZ1 e AZ2)
	- Min: 2, Desired: 2, Max: 4
	- Políticas de escalonamento (CPU > 70%)
	- Anexe o Load Balance target group

## Wordpress

- Acesse o ALB DNS name em um navegador
- Complete a configuração do WordPress
- Verifique:
	- Os uploads são persistentes nas instâncias (EFS)
	- A conexão com a base de dados RDS

