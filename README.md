# Instalação Wordpress básica em servidor Ubuntu 18.04 #

**1. Em uma instalação limpa do Ubuntu 18.04 faça um update do package manager e instale o Apache:**

```
sudo apt update
sudo apt install apache2
```
**2. Instale o Curl**
```
sudo apt install curl
```

**3. Instale o Mysql**
```
sudo apt install mysql-server
```
######Obs.: Caso vá usar banco em outro servidor não será necessário a instalação do mysql server.
**4. Dê uma senha para o usuário root do mysql**
```
sudo mysql
Select user,authentication_string,plugin,host FROM mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
**5. Rode o Flush Privileges para restartar a tabela de concessão do serviço mysql**
```
FLUSH PRIVILEGES;
exit
```
**6. Crie um banco de dados MYSQL e um USUÁRIO para o Wordpress**
```
mysql -u root -p
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
exit 
```
**7. Instale o PHP**
```
apt-get update
sudo apt-get install php-curl php-gd php-mbstring php-mcrypt php-xml php-xmlrpc
```
######  Obs.: Caso queira uma versão anterior do php utilize o número logo depois de php, exemplo : php7.2-curl php7.2-gd ... 

**8. Reinicie o apache**
```
sudo systemctl restart apache2
```
**9. Habilitar o Mod_rewrite para usar os permalinks do Wordpress**
```
sudo a2enmod rewrite
```
**10. Habilitar substituições .htaccess**
######Para permitir arquivos .htaccess, é necessário definir a diretiva AllowOverride dentro do bloco Directory apontando para a pasta raíz:
```
nano /etc/apache2/apache2.conf
```
>
>. . . 
><Directory /var/www/>
    AllowOverride All
</Directory>
>. . . 

Salve o arquivo com Ctrl+X + Y

**11. Habilite as alterações e reinicie**
```
sudo apache2ctl configtest
sudo systemctl restart apache2
```

**12. Configurando Hosts Virtuais** 

Dentro da pasta /var/www crie a pasta do site
```
sudo mkdir /var/www/NOMEDOSITE
```
Atribua a propriedade do diretório ao usuário ubuntu
```
sudo chown -R $USER:$USER /var /www/NOMEDOSITE
```
Crie um novo arquivo de conf em /etc/apache2/sites-available/NOMEDOSITE.conf
```
sudo nano /etc/apache2/sites-available/NOMEDOSITE.conf
```

 **/etc/apache2/sites-available/NOMEDOSITE.conf** 
 ```  
  <VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName NOMEDOSITE
    ServerAlias www.NOMEDOSITE
    DocumentRoot /var/www/NOMEDOSITE
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
 </VirtualHost>
```

Habilite o arquivo com o a2ensite

```
sudo a2ensite NOMEDOSITE.conf
```

Teste se existe algum erro de configuração:

```
sudo apache2ctl configtest
```

Estando tudo certo reinicie o apache

```
sudo systemctl restart apache2
```