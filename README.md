#!/bin/bash

# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Apache, PHP e dependências
sudo apt install -y apache2 wget php php-fpm php-mysql php-json php-dev

# Instalar MariaDB
sudo apt install -y mariadb-server

# Iniciar e habilitar serviços
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Configurar permissões para Apache
sudo usermod -a -G www-data ubuntu
sudo chown -R ubuntu:www-data /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;

# Clonar projeto
cd /var/www/html
git clone https://github.com/LindmanBassi/AulaAwsDesafio.git .

# Instalar OpenSSL e Apache Utils
sudo apt install -y openssl apache2-utils

# Criar diretórios para SSL
sudo mkdir -p /etc/ssl/private /etc/ssl/certs

# Criar certificado autoassinado
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt \
  -subj "/C=BR/ST=São Paulo/L=São Paulo/O=MinhaEmpresa/CN=localhost"

# Configurar SSL no Apache
sudo sed -i 's|SSLCertificateFile.*|SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt|' /etc/apache2/sites-available/default-ssl.conf
sudo sed -i 's|SSLCertificateKeyFile.*|SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key|' /etc/apache2/sites-available/default-ssl.conf

# Ativar site SSL e recarregar Apache
sudo a2ensite default-ssl
sudo systemctl reload apache2

# Baixar e extrair phpMyAdmin
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
mkdir phpMyAdmin
tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1

# Garantir que MariaDB está rodando
sudo systemctl start mariadb

echo "Setup concluído! Acesse https://SEU_IP para o site e http://SEU_IP/phpMyAdmin para o phpMyAdmin."
