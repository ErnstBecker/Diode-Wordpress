# Rodar o WordPress pela diode

Na seguinte documentação você encontrará como rodar o WordPress pela [diode](https://diode.io/).

## O que você encontrará?

Como instalar e Configurar

- LAMP
  - Apache
  - MariaDB
  - PHP
- WordPress
- Diode

### Instale o LAMP (Apache, MariaDB, PHP):

Instale de acordo com sua distribuição, talvez seja necessário fazer uma atualização antes das instalações:

<details>
	<summary>Debian/Ubuntu</summary>

```bash
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql
```

</details>
<br>
<details>
	<summary>Arch Linux</summary>

```bash
sudo pacman -S apache mariadb php php-apache php-mysql
```

</details>

### Configurar o Apache:

- Se o serviço não estiver rodando, inicie-o:
  ```bash
  sudo systemctl enable httpd
  sudo systemctl start httpd
  ```
- Edite o arquivo de configuração do PHP para que ele funcione com o Apache:
  ```bash
  sudo nano /etc/httpd/conf/httpd.conf
  ```
- Adicione ou descomente as seguintes linhas no final do arquivo:
  ```bash
  LoadModule php_module modules/libphp.so
  AddHandler php-script .php
  Include conf/extra/php_module.conf
  Include conf/extra/wordpress.conf
  ```
- Reinicie o Apache conforme o necessário:
  ```bash
  sudo systemctl restart httpd
  ```

### Configurar o MariaDB (MySQL):

- Inicie o MariaDB e configure a senha do root:
  ```bash
  sudo systemctl enable mariadb
  sudo systemctl start mariadb
  sudo mysql_secure_installation
  ```
- Crie um banco de dados chamado `wordpress`:
  ```bash
  sudo mysql -u root -p
  ```
  Dentro do MariaDB:
  ```sql
  CREATE DATABASE wordpress;
  CREATE USER 'nome_usuário'@'localhost' IDENTIFIED BY 'sua_senha';
  GRANT ALL PRIVILEGES ON wordpress.* TO 'nome_usuário'@'localhost';
  FLUSH PRIVILEGES;
  EXIT;
  ```
  Não esqueça de trocar o `nome_usuário` e a `sua_senha` pelas suas informações.

### Configurar o PHP:

- Certifique-se de que os módulos necessários para o WordPress estejam instalados:
  - php-mysql
  - php-apache
- Edite o arquivo /etc/php/php.ini para ativar o suporte ao MySQLi:
  ```bash
  sudo nano /etc/php/php.ini
  ```
- Descomente a linha:
  ```ini
  extension=mysqli
  ```
- Reinicie o Apache novamente:

  ```bash
  sudo systemctl restart httpd
  ```

- Pode ser necessário mudar o MPM para `mpm_prefork`
  O módulo `mpm_prefork` do Apache é mais adequado para rodar PHP, pois ele não é threaded e funciona melhor com módulos PHP que não são thread-safe.
  - Edite o arquivo de configuração do Apache para desativar o MPM atual (provavelmente `mpm_event`) e ativar o `mpm_prefork`:
    ```bash
    sudo nano /etc/httpd/conf/httpd.conf
    ```
  - Localize a linha que carrega o módulo `mpm_event` e comente-a (adicione um # no início da linha):
    ```conf
    #LoadModule mpm_event_module modules/mod_mpm_event.so
    ```
  - Ative o módulo `mpm_prefork`, removendo o comentário (ou adicionando-a caso ela não exista):
    ```conf
    LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
    ```

### Baixe o WordPress:

- Baixe a versão mais recente do WordPress:

  Para a instalação do WordPress você precisa do wget, verifique se você possui em sua distro ou instale de acordo com sua necessidade.

  ```bash
  wget https://wordpress.org/latest.tar.gz
  tar -xzvf latest.tar.gz
  ```

#### Mova o WordPress para o diretório web:

- Mova os arquivos do WordPress para o diretório padrão do Apache:
  ```bash
  sudo mv wordpress /srv/http/
  ```

#### Configurar permissões:

- Ajuste as permissões para o diretório do WordPress
  ```
  sudo chown -R http:http /srv/http/wordpress
  sudo chown -R 755 /srv/http/wordpress
  ```

#### Verifique a configuração do WordPress:

- Certifique-se de que o Apache está configurado corretamente para servir o WordPress: Você pode editar o arquivo de configuração do site:

  - Crie um novo arquivo de configuração no `/etc/httpd/conf/extra/wordpress.conf`

    ```bash
    sudo nano /etc/httpd/conf/extra/wordpress.conf
    ```

  - Adicione o seguinte conteúdo:

    ```bash
    <Directory "/srv/http/wordpress">
    	AllowOverride All
    	Require all granted
    </Directory>

    Alias /wordpress /srv/http/wordpress
    ```

  - Inclua este arquivo no `/etc/httpd/conf/httpd.conf`, adicionando a seguinte linha:

    ```bash
    Include conf/extra/wordpress.conf
    ```

	- Quando você iniciasse o servidor, iria perceber que ele iria apenas rodar em uma pasta e teria que acessar manualmente a pasta do WordPress, para resolver isso você pode já atribuir o caminho do WordPress no arquivo de configuração do `httpd.conf`
		```bash
		sudo nano /etc/httpd/conf/extra/wordpress.conf
		```

		Agora adicione ou substitua a parte onde diz `DocumentRoot` com o caminho do WordPress adicionando `/wordpress` como no exemplo abaixo:
		```conf
		...
		DocumentRoot "/srv/http/wordpress"
		<Directory "/srv/http/wordpress">
			Options Indexes FollowSymLinks
			AllowOverride All
			Require all granted
		</Directory>
		...
		```

  - Reinicie o Apache e o MariaDB conforme o necessário:

    ```bash
    sudo systemctl restart httpd
    sudo systemctl restart mariadb
    ```

#### Acesse o WordPress

- Acesse o WordPress em seu navegador:
  [http://localhost/wordpress](http://localhost/wordpress)
- Siga o processo de instalação, insira as informações do banco de dados (nome do banco:`wordpress`, usuário:`nome_usuário`, senha:`sua_senha`).

### Diode

- Para a instalação do diode, basta rodar o comando abaixo, ele irá fazer a instalação e mandar o caminho do executável para o /opt no / do seu sistema:

	```bash
	curl -Ssf https://diode.io/install.sh | bash && sudo mv ~/opt/diode /opt/ && rm -r ~/opt/ && export PATH=/opt/diode:$PATH
	```

- Para você abrir o WordPress no diode basta rodar o comando abaixo:

	```bash
	diode publish -public 80:80 -socksd
	```

- Caso você não saiba em que porta servidor Apache está rodando, você pode verificar no Listen no arquivo `/etc/httpd/conf/httpd.conf`

	```bash
	cat /etc/httpd/conf/httpd.conf
	```

No meu caso, ele estava na linha 52: `Listen 80`

#### Acesse o Link:

Pegue o link do seu site que estará no `INFO HTTP Gateway Enabled: http://0x....diode.link/`
