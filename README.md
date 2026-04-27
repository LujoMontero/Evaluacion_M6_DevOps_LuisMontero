<div align="center">

# ☁️ Plataforma VOD — Diseño de Infraestructura Cloud

### AWS · Terraform · Docker · ECS · GitHub Actions · GitFlow

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)

</div>

---

## 📌 Descripción del proyecto

Diseño de infraestructura cloud escalable para una **plataforma de Video Bajo Demanda (VOD)**, similar a Netflix. El proyecto define arquitectura AWS completa, pipeline CI/CD, estrategia de GitFlow e Infraestructura como Código con Terraform, capaz de soportar miles de usuarios simultáneos.

---

## 🏗️ Arquitectura Cloud (AWS)

```
                    ┌──────────────┐
   Usuarios ──────▶ │  CloudFront  │ ◀── S3 (videos + frontend estático)
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │     ALB      │  ← Application Load Balancer
                    └──────┬───────┘
                           │
              ┌────────────▼────────────┐
              │      ECS + Fargate      │  ← Backend en contenedores
              │    (Auto Scaling)       │
              └────────────┬────────────┘
                           │
                    ┌──────▼───────┐
                    │  RDS (PostgreSQL) │  ← Subred privada
                    └──────────────┘
```

### Servicios utilizados

| Capa | Servicio AWS | Propósito |
|---|---|---|
| Frontend | S3 + CloudFront | Distribución global de contenido estático |
| Cómputo | ECS + Fargate | Contenedores autogestionados y escalables |
| Base de datos | RDS PostgreSQL | Usuarios, catálogo y reproducciones |
| Almacenamiento | S3 + Glacier | Videos y backups a largo plazo |
| Balanceo | ALB | Distribución de tráfico |
| Seguridad | IAM + Secrets Manager + VPC | Roles, secretos y red privada |
| Monitoreo | CloudWatch + SNS | Métricas, logs y alertas |

---

## 🔒 Red y seguridad (VPC)

```
VPC
├── Subred pública   → Frontend, ALB, NAT Gateway
└── Subred privada   → ECS backend, RDS PostgreSQL
```

- **IAM Roles y Policies**: permisos mínimos por servicio
- **Security Groups + NACLs**: control granular de tráfico
- **Secrets Manager**: credenciales de BD y APIs sin hardcoding
- **VPN / IPSec**: acceso seguro a recursos internos

---

## ⚙️ Infraestructura como Código — Terraform

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

---

## 🔁 Pipeline CI/CD — GitHub Actions

```yaml
name: CI/CD — Plataforma VOD

on:
  push:
    branches: [main, "release/**"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install && npm test

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

---

## 🌿 Estrategia GitFlow

```
main ◀────────────── release/v1.0 ◀── QA ──── develop ◀── feature/perfil
  ▲                                                  ▲
  └──── hotfix/login (fix urgente)                   └── feature/streaming
```

- `main`: producción estable con tags de versión
- `develop`: integración continua de features
- `feature/*`: desarrollo aislado por funcionalidad
- `release/*`: estabilización antes de producción
- `hotfix/*`: correcciones urgentes en producción

---

## 📊 Monitoreo y alertas

| Herramienta | Función |
|---|---|
| CloudWatch | Métricas de CPU, memoria, latencia y errores HTTP |
| SNS | Notificaciones automáticas ante eventos críticos |
| ELK (alternativa) | Centralización y búsqueda de logs |

**Alertas configuradas:** caída de pods, errores 5xx, CPU > 80%, latencia > 1s

---

## 📊 Comparativa: Antes vs Después

| Aspecto | Antes | Después |
|---|---|---|
| Infraestructura | VPS inseguros sin gestión | AWS con VPC, IAM y Secrets Manager |
| Deploy | Manual por FTP | CI/CD automático con GitHub Actions |
| Escalabilidad | Manual y tardía | Auto Scaling Groups en ECS |
| Monitoreo | Sin métricas | CloudWatch + alertas SNS |
| Control de versiones | Sin repositorio | GitFlow en GitHub |
| IaC | Configuración manual | Terraform reproducible |

---

## 📄 Documentación completa

Disponible en: [`ArquitecturaCloud_M6_DevOps_LuisMontero.pdf`](./ArquitecturaCloud_M6_DevOps_LuisMontero.pdf)

---

## 👨‍💻 Autor

**Luis Montero** · Especialización DevOps · Julio 2025  
[GitHub](https://github.com/LujoMontero) · [LinkedIn](https://www.linkedin.com/in/luis-montero-if/)
