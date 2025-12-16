# Proyecto Terraform - Evaluación Final

Este proyecto contiene la infraestructura como código para desplegar una aplicación web con base de datos en AWS usando Terraform.

## 📋 Estructura del Proyecto

```
proyecto-terraform/
├── main.tf                     # Configuración principal
├── variables.tf                # Definición de variables
├── outputs.tf                  # Outputs del proyecto
├── backend.tf                  # Configuración del backend
├── terraform.tfvars.example    # Ejemplo de variables
├── modules/                    # Módulos reutilizables
│   ├── networking/             # Módulo de red (VPC, subnets, SG)
│   ├── compute/                # Módulo de instancias EC2
│   └── database/               # Módulo de base de datos RDS
└── docs/                       # Documentación adicional
```

## 🚀 Recursos Desplegados

### Networking
- **VPC**: Red virtual privada existente
- **Subnets**: 2 subredes públicas en diferentes AZ
- **Internet Gateway**: Acceso a internet
- **Route Tables**: Enrutamiento de tráfico
- **Security Groups**: Firewall para EC2 y RDS

### Compute
- **EC2 Instances**: 4 instancias (2 t2.micro, 2 t2.small)
- **Key Pair**: Para acceso SSH
- **AMI**: Amazon Linux 2 (más reciente)
- **User Data**: Configuración automática de Apache

### Database
- **RDS MySQL**: Base de datos gestionada
- **Subnet Group**: Para alta disponibilidad
- **Parameter Group**: Configuración optimizada
- **Secrets Manager**: Gestión segura de credenciales

## 📦 Prerrequisitos

1. **Terraform** >= 1.0
2. **AWS CLI** configurado
3. **Credenciales AWS** con permisos suficientes
4. **Par de claves SSH** generado

### Generar claves SSH (si no las tienes)
```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
```

## ⚙️ Configuración Inicial

### 1. Clonar y configurar
```bash
git clone <repository-url>
cd proyecto-terraform
```

### 2. Configurar variables
```bash
cp terraform.tfvars.example terraform.tfvars
# Editar terraform.tfvars con tus valores reales
```

### 3. Configurar backend (opcional pero recomendado)
```bash
# Crear bucket S3 para el estado
aws s3 mb s3://mi-terraform-state-bucket-unique-name

# Descomentar configuración en backend.tf
# Actualizar nombre del bucket
```

## 🚀 Despliegue

### Comandos básicos
```bash
# Inicializar Terraform
terraform init

# Validar configuración
terraform validate

# Formatear código
terraform fmt

# Planificar cambios
terraform plan

# Aplicar cambios
terraform apply

# Ver estado actual
terraform show

# Destruir recursos
terraform destroy
```

### Comandos de debugging
```bash
# Listar recursos en el estado
terraform state list

# Inspeccionar recurso específico
terraform state show aws_instance.app1

# Refrescar estado
terraform refresh

# Ver logs detallados
TF_LOG=DEBUG terraform plan
```

## 🔧 Uso de Módulos

### Módulo de Networking
```hcl
module "networking" {
  source = "./modules/networking"
  
  vpc_id      = var.vpc_id
  environment = var.environment
  
  subnet_configs = [
    {
      cidr   = "10.0.2.0/24"
      az     = "us-east-1a"
      public = true
    },
    {
      cidr   = "10.0.3.0/24"
      az     = "us-east-1b"
      public = true
    }
  ]
}
```

### Módulo de Compute
```hcl
module "compute" {
  source = "./modules/compute"
  
  environment         = var.environment
  instance_count      = var.instance_count
  subnet_ids          = module.networking.subnet_ids
  security_group_id   = module.networking.ec2_security_group_id
  key_pair_name       = var.key_pair_name
  public_key_path     = var.public_key_path
  private_key_path    = var.private_key_path
}
```

### Módulo de Database
```hcl
module "database" {
  source = "./modules/database"
  
  environment       = var.environment
  subnet_ids        = module.networking.subnet_ids
  security_group_id = module.networking.rds_security_group_id
  db_username       = var.db_username
  db_password       = var.db_password
}
```

## 🔍 Funciones de Terraform Utilizadas

### Funciones de Colección
- `length()`: Contar elementos en listas
- `merge()`: Combinar mapas
- `lookup()`: Buscar valores en mapas

### Funciones de String
- `format()`: Formatear strings
- `join()`: Unir elementos con separador
- `split()`: Dividir strings

### Funciones de Fecha
- `timestamp()`: Obtener timestamp actual
- `formatdate()`: Formatear fechas

### Funciones de Archivo
- `file()`: Leer contenido de archivo
- `templatefile()`: Procesar templates

### Expresiones Avanzadas
- `for_each`: Iterar sobre mapas/sets
- `count`: Crear múltiples recursos
- `dynamic`: Bloques dinámicos
- `locals`: Valores calculados

## 📊 Outputs Disponibles

Después del despliegue, obtendrás:

```bash
# IPs públicas de las instancias
terraform output instance_public_ips

# Comandos SSH
terraform output ssh_connection_commands

# URLs de los servidores web
terraform output web_urls

# Endpoint de la base de datos
terraform output database_endpoint
```

## 🔒 Seguridad

### Mejores Prácticas Implementadas
- ✅ Credenciales en variables sensibles
- ✅ Security Groups restrictivos
- ✅ RDS en subredes privadas
- ✅ Encriptación de almacenamiento
- ✅ Gestión de secretos con AWS Secrets Manager
- ✅ Validación de variables

### Consideraciones de Seguridad
- 🔐 Nunca hardcodear credenciales
- 🔐 Usar IAM roles cuando sea posible
- 🔐 Restringir acceso SSH por IP
- 🔐 Habilitar logs de CloudWatch
- 🔐 Usar HTTPS en producción

## 🐛 Troubleshooting

### Errores Comunes

**Error: Invalid AMI ID**
```bash
# Verificar AMIs disponibles
aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*"
```

**Error: Insufficient permissions**
```bash
# Verificar permisos IAM
aws sts get-caller-identity
aws iam list-attached-user-policies --user-name tu-usuario
```

**Error: VPC not found**
```bash
# Verificar VPCs disponibles
aws ec2 describe-vpcs
```

**Error: Key pair not found**
```bash
# Listar key pairs existentes
aws ec2 describe-key-pairs
```

### Logs y Debugging
```bash
# Habilitar logs detallados
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log

# Ver logs de instancias EC2
aws logs get-log-events --log-group-name /aws/ec2/user-data
```

## 📚 Recursos Adicionales

- [Documentación de Terraform](https://www.terraform.io/docs)
- [Provider AWS](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Mejores Prácticas](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [Funciones de Terraform](https://www.terraform.io/docs/language/functions/index.html)

## 🤝 Contribución

1. Fork el proyecto
2. Crear rama feature (`git checkout -b feature/nueva-funcionalidad`)
3. Commit cambios (`git commit -am 'Agregar nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Crear Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para detalles.

---

**¡Éxito en tu evaluación!** 🚀

Para soporte adicional, consulta la documentación en la carpeta `docs/` o contacta al instructor.