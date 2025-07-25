# Evaluación Módulo 6: Diseño de Infraestructura Cloud y DevOps para una Plataforma de Video Bajo Demanda

## 🌟 Contexto

Una empresa de tecnología desea crear una plataforma de video bajo demanda (VOD), similar a Netflix.  
Los usuarios podrán registrarse, seleccionar contenido y hacer streaming desde diversos dispositivos.  
Se requiere una infraestructura cloud escalable, segura y eficiente, capaz de atender a miles de usuarios simultáneos.

---

## ✅ Objetivo del Informe

Diseñar una infraestructura cloud moderna para la plataforma VOD, justificando cada elección respecto a:

- Servicios de almacenamiento.
- Cómputo y escalabilidad.
- Redes y seguridad.
- CI/CD e infraestructura como código.

---

## 1. Diseño de la Infraestructura en la Nube

<img width="1536" height="1024" alt="diagrma de flujo" src="https://github.com/user-attachments/assets/9fa10dba-5f7b-44f5-8dea-6a7f21013524" />

---
### ✈ Modelo de implementación

- Tipo: **Nube pública (AWS)**
- Justificación:
  - Alta disponibilidad global.
  - Red de entrega de contenido (CDN).
  - Reducción de costos en mantenimiento y hardware.

### 🚀 Modelo de servicio

- Modelo híbrido:
  - **IaaS**: EC2 y redes
  - **PaaS**: RDS
  - **FaaS**: Lambda para tareas programadas o lógica liviana

### 📂 Almacenamiento

- S3: contenido multimedia estático (videos)
- Glacier: backups y almacenamiento a largo plazo
- RDS PostgreSQL: base de datos relacional (usuarios, reproducciones, catálogo)

---

## 2. Arquitectura de Cómputo y Escalabilidad (1.5 pt)

### ⚖️ Elección de servicios

- ECS con Fargate: backend en contenedores autogestionados
- S3 + CloudFront: frontend estático distribuido globalmente

### 🚧 Escalabilidad

- Auto Scaling Groups (EC2 o ECS)
- Application Load Balancer (ALB)

### 🌐 Orquestador

- Amazon ECS (alternativa: EKS - Kubernetes)

---

## 3. Redes y Seguridad en la Nube (1.5 pt)

### 🛫 Red (VPC)

- Subredes separadas:
  - Pública (frontend, balanceador)
  - Privada (RDS, ECS)
- Tablas de ruteo y Gateway NAT

### 🚦 Seguridad

- IAM Roles y Policies
- Security Groups y NACLs
- Secrets Manager
- VPN e IPSec

### 🌎 CDN

- Amazon CloudFront

---

## 4. Automatización e Integración Continua (1.5 pt)

### ⚒️ CI/CD Pipeline (GitHub Actions)

```yaml
name: CI/CD - Plataforma VOD

on:
  push:
    branches: [ "main", "release/**" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ec2-user/vod-backend
            git pull origin main
            npm install
            pm2 restart app.js
```
## 📁 Infraestructura como Código (Terraform)

Ejemplo de recurso en Terraform para una instancia EC2:

```hcl
resource "aws_instance" "vod_backend" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public_subnet.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              docker run -d -p 80:80 nginx
              EOF
}

```
## 📊 Monitoreo

- **AWS CloudWatch**: Para monitoreo de logs, métricas y alertas.
- **Amazon SNS**: Para envío de notificaciones automáticas ante eventos críticos.

---

## 🕓 Antes y Después del Ejercicio

### ❌ Antes:

- Servidores VPS inseguros  
- Deploy manual (por FTP)  
- Sin pipelines  
- Sin monitoreo  
- Sin control de versiones  

### ✅ Después:

- Infraestructura AWS bien estructurada (VPC, S3, RDS, ECS)  
- CI/CD con GitHub Actions  
- Terraform usado como Infraestructura como Código (IaC)  
- Flujo de trabajo GitFlow aplicado  

## 📑 GitFlow (Mermaid)

```mermaid
gitGraph
   commit id: "Inicio"
   branch develop
   checkout develop
   commit id: "Desarrollo inicial"
   branch feature/perfil
   checkout feature/perfil
   commit id: "Agrega perfiles"
   checkout develop
   merge feature/perfil
   branch release/v1.0
   commit id: "QA"
   checkout main
   merge release/v1.0 tag: "v1.0"
   branch hotfix/login
   commit id: "Fix login"
   merge hotfix/login
```
<img width="3840" height="3160" alt="Untitled diagram _ Mermaid Chart-2025-07-25-023258" src="https://github.com/user-attachments/assets/1b9494d1-fe85-4694-b037-5ff106326a2c" />

---

##📊 Herramientas Utilizadas

- Terraform  
- GitHub Actions
- AWS (EC2, S3, ECS, RDS, CloudWatch, IAM)
- Draw.io (para el diagrama)
- Mermaid (para visualizar el flujo GitFlow)
- Node.js + PM2 (para ejecución del backend)
