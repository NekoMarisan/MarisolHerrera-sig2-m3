# Magento 2 - Instalación en Garuda Linux con Docker + Mark Shust + Composer

Este documento detalla el proceso para instalar Magento 2.4.4 en Garuda Linux usando el stack de **Mark Shust (docker-magento)** y gestionando las dependencias con **Composer**.


## Requisitos del Sistema

### Equipo

- **OS:** Garuda Linux (basado en Arch Linux)
- **Espacio en Disco:** ≥ 10 GB
- **RAM:** ≥ 8 GB


## Instalación paso a paso

### 1. Actualizar sistema

```
sudo pacman -Syu
```

### 2. Instalar dependencias básicas

```
sudo pacman -S --needed git curl wget unzip vim docker docker-compose base-devel
```

### 3. Habilitar Docker

```
sudo systemctl enable docker.service
sudo systemctl start docker.service
```

### 4. Añadir tu usuario al grupo \`docker\`

```
sudo usermod -aG docker \$USER
newgrp docker
```

### 5. Verificar Docker y Docker Compose

```
docker --version
docker-compose --version
```

### 6. Instalar Composer

```
cd ~
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
```

Verificar:

```
composer --version
```


## Llaves de Magento Marketplace

1. Crear cuenta en [https://account.magento.com](https://account.magento.com)
2. Ir a **Access Keys** y generar llaves
3. Guardarlas para uso posterior con Composer


## Preparación del proyecto

### 7. Crear directorio de trabajo

```
mkdir -p ~/Sites/magento
cd ~/Sites/magento
```

## Instalación de Magento con Mark Shust docker-magento

### 8. Ejecutar instalador

```
curl -s https://raw.githubusercontent.com/markshust/docker-magento/master/lib/onelinesetup | bash -s -- magento.test 2.4.4 community
```

### 9. Configurar \`/etc/hosts\`

```
sudo nano /etc/hosts
```

Agregar:

```
127.0.0.1 magento.test
```

### 10. Acceder en navegador

- [http://magento.test](http://magento.test)
- [http://magento.test/admin](http://magento.test/admin)


## Gestión de contenedores

```
# Detener contenedores
bin/stop

# Iniciar contenedores
bin/start
```


## Crear usuario admin en Magento

```
bin/magento admin:user:create --admin-user="admin" --admin-password="Admin1234" --admin-email="admin@example.com" --admin-firstname="Admin" --admin-lastname="User"
```


## Desactivar autenticación de dos factores

```
bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth
bin/magento module:disable Magento_TwoFactorAuth
bin/magento setup:upgrade && bin/magento setup:di:compile
```


## Resetear instalación (opcional)

Si hay problemas en la instalación:

```
bin/stop
cd ..
rm -rf ~/Sites/magento/{*,.*} 2>/dev/null
mkdir ~/Sites/magento
cd ~/Sites/magento
curl -s https://raw.githubusercontent.com/markshust/docker-magento/master/lib/onelinesetup | bash -s -- magento.test 2.4.4 community
```


## Configuración de Git + SSH

### 1. Configuración global de Git

```
git config --global user.name "tu_nombre"
git config --global user.email "tu_correo@ejemplo.com"
git config --list
```

### 2. Generar clave SSH

```
ssh-keygen -t ed25519 -C "tu_correo@ejemplo.com"
eval "\$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Agregar clave en: [https://github.com/settings/keys](https://github.com/settings/keys)

### 3. Verificar conexión SSH

```
ssh -T git@github.com
```


## Composer y permisos

### 1. Instalar dependencias con Composer

```
bin/composer install
bin/composer update
```

### 2. Revisar y corregir permisos

```
id -u
id -g

# Asignar permisos correctos
sudo chown -R \$(whoami):docker ~/Sites/magento
bin/fixowns
bin/fixperms
```


## Comandos útiles de Magento

```
# Setup Upgrade
bin/magento setup:upgrade

# Deploy static content
bin/magento setup:static-content:deploy -f

# Reindex
bin/magento indexer:reindex
```


## Finalización

Acceder a:

- [http://magento.test](http://magento.test)
- [http://magento.test/admin](http://magento.test/admin)


## Notas adicionales

- Composer gestiona dependencias y módulos adicionales.
- Docker stack de Mark Shust facilita configuración de Magento en contenedores.
- PHP versión recomendada para Magento 2.4.4: **PHP 8.1.x**.
- En Garuda Linux no se requiere WSL como en Windows.


## Créditos

Basado en la guía original adaptada para uso en Garuda Linux.

