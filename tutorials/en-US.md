# Run Wordpress on diode

In the following documentation you will find how to run Wordpress on [diode](https://diode.io/).

## What you will find?

How to install and Configure

- LAMP
  - Apache
  - MariaDB
  - PHP
- WordPress
- Diode

### Install LAMP (Apache, MariaDB, PHP):

Install according to your distribution, maybe you need to do an update before the installations:

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

### Configure Apache:

- If the service is not running, start it:
  ```bash
  sudo systemctl enable httpd
  sudo systemctl start httpd
  ```
- Edit the PHP configuration file to make it work with Apache:
  ```bash
  sudo nano /etc/httpd/conf/httpd.conf
  ```
- Add or comment the following lines at the end of the file:
  ```bash
  LoadModule php_module modules/libphp.so
  AddHandler php-script .php
  Include conf/extra/php_module.conf
  Include conf/extra/wordpress.conf
  ```
- Restart Apache as needed:
  ```bash
  sudo systemctl restart httpd
  ```

### Configure MariaDB (MySQL):

- Start MariaDB and configure the root password:
  ```bash
  sudo systemctl enable mariadb
  sudo systemctl start mariadb
  sudo mysql_secure_installation
  ```
- Create a database called `wordpress`:
  ```bash
  sudo mysql -u root -p
  ```
  Inside MariaDB:
  ```sql
  CREATE DATABASE wordpress;
  CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
  GRANT ALL PRIVILEGES ON wordpress.* TO 'username'@'localhost';
  FLUSH PRIVILEGES;
  EXIT;
  ```
  Don't forget to change the `username` and `password` to your own.

### Configure PHP:

- Make sure that the required modules for WordPress are installed:
  - php-mysql
  - php-apache
- Edit the /etc/php/php.ini file to enable MySQLi support:
  ```bash
  sudo nano /etc/php/php.ini
  ```
- Comment out the line:
  ```ini
  extension=mysqli
  ```
- Restart Apache again:

  ```bash
  sudo systemctl restart httpd
  ```

- It may be necessary to change the MPM to `mpm_prefork`
  The `mpm_prefork` module for Apache is more appropriate to run PHP, as it is not threaded and works better with PHP modules that are not thread-safe.
  - Edit the Apache configuration file to disable the current MPM (probably `mpm_event`) and enable the `mpm_prefork`:
    ```bash
    sudo nano /etc/httpd/conf/httpd.conf
    ```
  - Locate the line that loads the `mpm_event` module and comment it out (add a # at the beginning of the line):
    ```conf
    #LoadModule mpm_event_module modules/mod_mpm_event.so
    ```
  - Enable the `mpm_prefork` module, removing the comment (or adding it if it doesn't exist):
    ```conf
    LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
    ```

### Download WordPress:

- Download the latest version of WordPress:

  To install WordPress you need wget, check if you have it in your distro or install it according to your needs.

  ```bash
  wget https://wordpress.org/latest.tar.gz
  tar -xzvf latest.tar.gz
  ```

#### Move WordPress to the web directory:

- Move the WordPress files to the default Apache directory:
  ```bash
  sudo mv wordpress /srv/http/
  ```

#### Set permissions:

- Adjust the permissions for the WordPress directory
  ```
  sudo chown -R http:http /srv/http/wordpress
  sudo chown -R 755 /srv/http/wordpress
  ```

#### Check the WordPress configuration:

- Make sure that the Apache server is configured correctly to serve WordPress: You can edit the site configuration file:

  - Create a new configuration file in `/etc/httpd/conf/extra/wordpress.conf`

    ```bash
    sudo nano /etc/httpd/conf/extra/wordpress.conf
    ```

  - Add the following content:

    ```bash
    <Directory "/srv/http/wordpress">
    	AllowOverride All
    	Require all granted
    </Directory>

    Alias /wordpress /srv/http/wordpress
    ```

  - Include this file in `/etc/httpd/conf/httpd.conf`, adding the following line:

    ```bash
    Include conf/extra/wordpress.conf
    ```

	- When you start the server, you will notice that it will only run in a folder and you will have to manually access the WordPress folder to resolve this, so you can already assign the WordPress path in the `httpd.conf` configuration file
		```bash
		sudo nano /etc/httpd/conf/extra/wordpress.conf
		```

		Now add or replace the part where it says `DocumentRoot` with the WordPress path adding `/wordpress` as in the example below:
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

  - Restart Apache and MariaDB as needed:

    ```bash
    sudo systemctl restart httpd
    sudo systemctl restart mariadb
    ```

#### Access WordPress

- Access WordPress on your browser:
  [http://localhost/wordpress](http://localhost/wordpress)
- Follow the installation process, enter the database information (database name:`wordpress`, username:`username`, password:`password`).

### Diode

- To install diode, just run the command below, it will install and send the executable path to the /opt in the / of your system:

	```bash
	curl -Ssf https://diode.io/install.sh | bash && sudo mv ~/opt/diode /opt/ && rm -r ~/opt/ && export PATH=/opt/diode:$PATH
	```

- To open WordPress on diode just run the command below:

	```bash
	diode publish -public 80:80 -socksd
	```

- If you don't know what port the Apache server is running on, you can check the Listen in the `/etc/httpd/conf/httpd.conf` file

	```bash
	cat /etc/httpd/conf/httpd.conf
	```

In my case, it was on line 52: `Listen 80`

#### Access Link:

Get the link of your site that will be on `INFO HTTP Gateway Enabled: http://0x....diode.link/`
