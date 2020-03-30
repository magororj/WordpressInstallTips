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

**13. Baixe ou copie o projeto wordpress para a pasta /var/www/**

Para o caso de uma instalação nova:
```
cd /tmp
curl -O https://br.wordpress.org/latest-pt_BR.tar.gz 
```
Extraia o arquivo e mova para a pasta /www
```
tar xzvf latest-pt_BR.tar.gz
sudo cp -a /tmp/wordpress/. /var/www/NOMEDOSITE
```
**15. Ajuste a Propriedade e permissões**
```
sudo chown -R ubuntu:www-data /var/www/
sudo find /var/www/NOMEDOSITE -type d -exec chmod g+s {} \;
```
Ajustando permissão de escrita para o grupo para o diretório /wp-content 
```
sudo chmod g+w /var/www/NOMEDOSITE/wp-content
```

Como parte desse processo, daremos ao servidor web acesso de escrita a todo o conteúdo nesses dois diretórios:
``` 
sudo chmod -R g+w /var/www/NOMEDOSITE/wp-content/themes
sudo chmod -R g+w /var/www/NOMEDOSITE/wp-content/plugins
```

**16. Ajuste o wp-config.php**
```
/var/www/NOMEDOSITE/wp-config.php
```
```
. . .

define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');

. . .

define('FS_METHOD', 'direct');
```
**17. Complete a instalação através da interface web**

Ajuste permissões para atualizar
Quando uma atualização se torna disponível, acesse novamente seu servidor como usuário do sudo. Temporariamente conceda ao processo do servidor web, acesso à pasta raiz inteira.
```
sudo chown -R www-data /var/www/html
```
Volte ao painel de administração do Wordpress e aplique a atualização.
Quando tiver terminado, volte com as permissões anteriores novamente:
```
sudo chown -R ubuntu /var/www/html
```
Isso deve ser necessário apenas quando você estiver aplicando atualizações ao próprio WordPress.

**18. Continue com as recomendações de segurança abaixo.** 

## Principais Recomendações de Segurança ##

**1. Utilizar a versão mais recente do PHP.**

A partir da  versão 7 do PHP somente os Branch com até dois anos terão suporte em atualização de bugs e segurança, por exemplo a versão 7.3 lançada em dezembro de 2018 terá suporte de segurança até dezembro de 2021.

**2. Usar nomes de usuários e senhas inteligentes**

Utilizar logins e senhas únicas para cada site além das ferramentas de criação de logins fortes que o próprio Wordpress disponibiliza. 

**3. Atualize sempre as versões mais recentes do Wordpress, Plugins e temas** 

Sempre que for utilizar um plugin ou tema que não seja do repositório do Wordpress, utilizar uma ferramenta on-line de scaneamento de malware como o [Virus Total](https://www.virustotal.com/gui/home/upload).

**4. Bloquear a área de administração do Wordpress**

* Utilizar um nome alternativo para o endereço de administração (wp-admin); 
* Forçar a autenticação de dois fatores;
* Limitar tentativas por IP;
* Utilizar autenticação básica por htpasswd;
* Restringir o acesso a administração somente de um IP conhecido

**5. Reforce a segurança do wp-config.php**

O __wp-config.php__ como o nome sugere é o principal arquivo de configuração de um sistema Wordpress, contém informações de login e banco de dados extremamente sensíveis, além das chaves de segurança que manipulam a criptografia das informações nos cookies. 
As principais ações no que tange a segurança deste arquivo são:
* Mover o wp-config.php da raiz do projeto, procure mantê-lo um diretório acima do diretório raiz;
* Atualizar as chaves de segurança; sempre que necessário utilize : 
```
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
* Alterar as permissões para 600 


**6. Utilizar plugins de segurança do Wordpress**

As principais soluções :

* Sucuri Security
* iThemes Security
* WordFence Security
* WP fail2ban
* SecuPress

**Principais funções:** 

* Gerar e forçar senhas fortes ao criar perfis de usuário
* Forçar que as senhas expirem e sejam redefinidas regularmente
* Registro de ações do usuário
* Atualizações fáceis de chaves de segurança do WordPress
* Análise de Malware
* Autenticação de dois fatores
* reCAPTCHAs
* Firewalls de segurança do WordPress
* Lista de permissões de IP
* Lista negra de IP
* Logs de alteração de arquivo
* Monitorar alterações de DNS
* Bloquear redes maliciosas
* Ver informações WHOIS sobre visitantes

>Talvez a característica mais importante e por que se deve usar um plugin de segurança no Wordpress é que a maioria utiliza um utilitário de soma de verificação (Checksums). Através da comparação com os arquivos core do Wordpress.org o Plugin varre o ambiente em busca de arquivos ou pastas  alteradas ou modificadas.  

**7. Atenção especial ao banco de dados**

* Atenção ao prefixo do banco  (renomeie sempre o wp_ por outro prefixo mais seguro)
* Se possível crie um usuário de banco específico para o ambiente com permissões restritas somente aquele banco, JAMAIS use o root do banco.
* Execute ou tenha scripts de back-up rotineiramente.

**8. Permissões do servidor*

As recomendações típicas para permissões são:

* Todos os arquivos devem ser 644 ou 640. Exceção: o wp-config.php e o debug.log devem ser 600.
* Todos os diretórios devem ser 755 ou 750.
* Nenhum diretório deveria receber 777, nem mesmo fazer o upload de diretórios.
* O melhor a fazer é restringir ao máximo as permissões dos arquivos e pastas e ir liberando apenas os casos que necessitem de permissão de escrita como a pasta Upload.
* A sua conta de usuário (Ubuntu, por exemplo) deve ser a dona de todos os arquivos, tendo acesso de escrita a todos eles. Os arquivos que necessitem de escrita pelo Wordpress (www-data no Debian) devem estar em um grupo, também pertencente a conta de usuário utilizada no servidor.

Novamente o iThemes Security possui um utilitário para verificar se as permissões no seu site estão corretas.

![Permissões](https://github.com/magororj/WordpressInstallTips/blob/master/image2.png)