# Setup WordPress Website Using LAMP Stack

Setting up a WordPress website using the LAMP stack (Linux, Apache, MySQL, PHP) is a powerful way to create a robust and scalable web presence. This project will guide you through each step of the process, from setting up your server environment to installing WordPress and configuring it for optimal performance. Whether you're a seasoned developer or a beginner, this tutorial will provide you with the knowledge and skills needed to deploy a fully functional WordPress site. By the end of this guide, you'll have a solid understanding of how to leverage the LAMP stack to build and manage your own WordPress website, ensuring a secure, efficient, and customizable platform for your content.


## Documentation

Please reference [**Project1**](https://github.com/pinea1111/project-1-docu/blob/master/project1docu.md) for guidance on spinning up an Ubuntu server.

- Set an inbound rule for MYSQL in your security group. Click on **Security** and select the **Security group**.

![1](img/Screenshot%20(102).png)

- click on **edit inbound rules**.

![2](img/Screenshot%20(103).png)

- clicl on **add rules**

![3](img/Screenshot%20(104).png)

- Click on **Custom TCP** and select **MySQL/Aurora**.

- Enter the **IP address** you want to allow access and click **Save rules**

> [!NOTE]
You can choose 0.0.0.0/0 to permit access from anywhere, but for added security, restrict access to MySQL exclusively to your Web Server’s IP address. In the Inbound Rule configuration, specify the source as /32

- Open your **terminal** and connect to your Ubuntu server via SSH.

![5](img/Screenshot%20(105).png)

### Install Apache

- To install Apache, run the following commands in your terminal.

**`sudo apt update`**

![6](img/Screenshot%20(106).png)

**`sudo apt install apache2`**

![7](img/Screenshot%20(107).png)

- To enable Apache to start on boot, execute **`sudo systemctl enable apache2`**, and then verify its status with the **`sudo systemctl status apache2`** command.

![8](img/Screenshot%20(108).png)

- Now, let's check if our server is running and accessible both locally and from the Internet by executing the following command: **`curl http://localhost:80`**.

![9](img/Screenshot%20(109).png)

*Now, it's time to test how our Apache HTTP server responds to requests from the Internet.*

- Copy your **public IPv4 address** from your EC2 dashboard.

- Open a web browser of your choice and try accessing the following URL: **`http://<Public-IP-Address>:80`**

- If the installation was successful, you should see this page.

![10](img/Screenshot%20(110).png)

---

### Install MYSQL

- To install this software using 'apt', run the command **`sudo apt install mysql-server`**. When prompted, confirm the installation by typing 'Y' and then pressing ENTER.

![11](img/Screenshot%20(111).png)

- After the installation is complete, log in to the MySQL console by typing: **`sudo mysql`**.

![12](img/Screenshot%20(112).png)


> [!NOTE]
It's recommended that you run a security script included with MySQL to enhance security. Before running the script, set a password for the root user using the default authentication method **mysql_native_password**. For this project, we'll define the password as "**pass**", but you can choose any password you prefer.

- Run the following command to set the password for the root user with the MySQL native password authentication method: **`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'pass';`**. Exit the MySQL shell when you're done by typing **`exit`**.

![13](img/Screenshot%20(114).png)

- Start the interactive script by running: **`sudo mysql_secure_installation`**. Answer **y** for yes, or any other key to continue without enabling specific options.

![14](img/Screenshot%20(115).png)

- Set your **password validation policy level**.

> [!NOTE]
I set my password validation policy level to 0 because I don't require much security, as I will be terminating all resources immediately after this project. However, on the job, it's advised to use the strongest level, which is 2.

- Enable MySQL to start on boot by executing **`sudo systemctl enable mysql`**, and then confirm its status with the **`sudo systemctl status mysql`** command.

![15](img/Screenshot%20(116).png)

---

### Install PHP

- Install PHP along with required extensions by running the following script: **`sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip`**.

![16](img/Screenshot%20(117).png)

**`sudo apt install php libapache2-mod-php php-mysql`**

![17](img/Screenshot%20(118).png)

- Confirm the downloaded PHP version by running **`php -v`**.

![18](img/Screenshot%20(119).png)

### Creating A Virtual Host For Your Website Using Apache

- Create the directory for Projectlamp using the 'mkdir' command as follows:
**`sudo mkdir /var/www/projectlamp`①** and assign ownership of the directory to our current system user using:
**`sudo chown -R $USER:$USER /var/www/projectlamp`②**.

![19](img/Screenshot%20(120).png)

- Create and open a new configuration file in Apache's sites-available directory using your preferred command-line editor:
**`sudo vi /etc/apache2/sites-available/projectlamp.conf`**.

- Creating this will produce a new blank file. Paste the configuration text provided below into it:

```
<VirtualHost *:80>

ServerName projectlamp

ServerAlias www.projectlamp

ServerAdmin webmaster@localhost

DocumentRoot /var/www/projectlamp

ErrorLog ${APACHE_LOG_DIR}/error.log

CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

- Save your changes by pressing the **`Esc`** key, then type **`:wq`** and press **`Enter`**.

- Run the ls command **`sudo ls /etc/apache2/sites-available`①** to show the **new file②** in the sites-available directory.

![20](img/Screenshot%20(121).png)

- We can now enable the new virtual host using the a2ensite command: **`sudo a2ensite projectlamp`**.

![21](img/Screenshot%20(122).png)

- To disable Apache's default website, use the a2dissite command. Type: **`sudo a2dissite 000-default`**.

![22](img/Screenshot%20(123).png)

- To ensure your configuration file doesn’t contain syntax errors, run: **`sudo apache2ctl configtest`**. You should see **"Syntax OK"** in the output if your configuration is correct.

![23](img/Screenshot%20(124).png)

- Finally run: **`sudo systemctl reload apache2`**. This will reload Apache for the changes to take effect.

> [!NOTE]
Our new website is now active, but the web root **`/var/www/projectlamp`** is still empty. Let's create an **`index.html`** file in that location to test that the virtual host works as expected.

- To create the **index.html** file with the content **"Hello LAMP from Jay"** in the /var/www/projectlamp directory, use the following command: **`sudo echo 'Hello LAMP from Jay' > /var/www/projectlamp/index.html`**.

![24](img/Screenshot%20(125).png)

- Now, let's open our web browser and try to access our website using the IP address:

**`http://<EC2-Public-IP-Address>:80`**

> [!NOTE]
Replace **`<EC2-Public-IP-Address>`** with your actual EC2 instance's public IP address.

![25](img/Screenshot%20(126).png)

- Remove the index.html file by running the following command: **`sudo rm /var/www/projectlamp/index.html `**

### Enable PHP On The Website

With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file. To change the precedence of index files (such as index.php over index.html) in Apache, you'll need to edit the dir.conf file. Here’s how you can do it:

- Edit the dir.conf file using a text editor (such as nano or vi): **`sudo nano /etc/apache2/mods-enabled/dir.conf`**

- Look for the DirectoryIndex directive within this file. It typically looks like this:

```
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```

- To prioritize **index.php** over **index.html**, move **index.php** to the beginning of the list, like this:

```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

- Press **`ctrl` + `x`** on your keyboard to save and exit.

- Type **`y`** to save the changes.

- When prompted to confirm the file name, simply press **`ENTER`** to save the changes with the existing file name.

- Finally, reload Apache for the changes to take effect: **`sudo systemctl reload apache2`**.

Now, Apache will prioritize index.php over index.html when both files exist in the same directory.

- To create a new file named index.php inside your custom web root folder (/var/www/projectlamp), you can use the following command to open it in the nano text editor: **`nano /var/www/projectlamp/index.php`**.

- This will create a new file. Copy and paste the following PHP code into the new file:

```
<?php

phpinfo();
```
![26](img/Screenshot%20(127).png)

- Once you've saved and closed the file, go back to your web browser and refresh the page. You should see something like this:

![27](img/Screenshot%20(128).png)

> [!NOTE]
This page provides information about your server from the perspective of PHP. It is useful for debugging and to ensure that your settings are being applied correctly.

If you can see this page in your browser, then congratulations🎉 your PHP installation is working as expected.
After verifying the relevant information about your PHP server through that page, it's recommended to remove the file you created, as it contains sensitive information about your PHP environment and your Ubuntu server. You can use the rm command to do so:
**`sudo rm /var/www/projectlamp/index.php`**.

You can always recreate this page if you need to access the information again later.

---
### Install Wordpress

After setting up our LAMP environment, we can start installing WordPress. First, we'll download the WordPress installation files and place them in the default web server root directory: **/var/www/html**.

- Navigate to the directory using the cd command **`cd /var/www/html`**, and then download the WordPress installation files using the following command: **`sudo wget -c http://wordpress.org/latest.tar.gz`**

![28](img/Screenshot%20(129).png)

- Extract the files from the downloaded WordPress archive using the command: **`sudo tar -xzvf latest.tar.gz`**

![29](img/Screenshot%20(130).png)

- Run the command **`ls -l`** to confirm the existence of the **wordpress** directory in the current location (`/var/www/html`).

![30](img/Screenshot%20(131).png)

> [!NOTE]
The files must be owned by the user of your web server. Identify the web server's user and assign the appropriate permissions accordingly. The user **`www-data`** is widely adopted as the default user for web server processes, especially on Ubuntu and Debian systems. This user oversees the operation of web server software (like Apache or Nginx) and manages incoming web requests. However, for verification and for tasks involving services that may not have a predefined user, checking the user of the web server is advisable.

- Check the user running your web server with the command: **`ps aux | grep apache | grep -v grep`**.

![31](img/Screenshot%20(132).png)

*This command filters processes related to Apache (apache2 on Ubuntu) and displays information about the user running those processes.*

- Grant ownership of the WordPress directory and its files to the web server user **(www-data)** by running the command: **`sudo chown -R www-data:www-data /var/www/html/wordpress`**.

![32](img/Screenshot%20(133).png)

### Create a Database For Wordpress

- Access your MySQL root account with the following command: **`sudo mysql -u root -p`①**. Enter the **password②** you set earlier when prompted.

- To create a separate database named wp_db for WordPress to manage, execute the following command in the MySQL prompt: **`CREATE DATABASE wp_db;`**

![33](img/Screenshot%20(133.1).png)

> [!NOTE]
This command allows you to create a new database (**wp_db**) within your MySQL environment. Feel free to name it as you prefer.

- To access the new database, you can create a MySQL user account with a strong password using the following command:
**`CREATE USER pinea@localhost IDENTIFIED BY 'wp-password';`**

![34](img/Screenshot%20(134).png)

- To grant your created user (pinea@localhost) all privileges needed to work with the wp_db database in MySQL, use the following commands:

```
GRANT ALL PRIVILEGES ON wp_db.* TO pinea@localhost;
FLUSH PRIVILEGES;
```
![35](img/Screenshot%20(136).png)

> [!NOTE]
This grants all privileges **(ALL PRIVILEGES)** on all tables within the wp_db database **(`wp_db.*`)** to the user pinea when accessing from localhost. The FLUSH PRIVILEGES command ensures that MySQL implements the changes immediately. Adjust the database name **(wp_db)** and username **(pinea)** as per your setup.

- Type **`exit`** to exit the MySQL shell.

![36](img/Screenshot%20(137).png)

- Grant executable permissions recursively (-R) to the wordpress folder using the following command: **`sudo chmod -R 777 wordpress/`**

> [!NOTE]
This command sets read (r), write (w), and execute (x) permissions for the owner, group, and others on all files and directories within the wordpress folder. Using 777 permissions is quite permissive and may not be necessary for all files and folders; consider adjusting permissions based on security requirements.

- Change into the WordPress directory by running the command: **`cd wordpress`**.

![37](img/Screenshot%20(139).png)

### Configure Wordpress

Once you've established a database for WordPress, the next crucial step is setting up and configuring WordPress itself. To begin, you'll need to create a configuration file tailored for WordPress.

- Rename the sample WordPress configuration file with the command: **`mv wp-config-sample.php wp-config.php`**.

- Edit the **`wp-config.php`** file using the command: **`sudo nano wp-config.php`**.

- Update the database settings in the **`wp-config.php`** file by replacing placeholders like **database_name_here**, **username_here**, and **password_here** with your actual database details.


![38](img/Screenshot%20(140).png)

- Modify the configuration file projectlamp.conf: **`sudo nano /etc/apache2/sites-available/projectlamp.conf`** to update the document root to the directory where your WordPress installation is located.

- After updating the document root to **`/var/www/html`** directory in your editor, save the changes and exit.

- Reload Apache for the changes to take effect: **`sudo systemctl reload apache2`**.

- Once you've completed these steps, you can access your WordPress page to complete the installation. Open your web browser and go to **`http://<EC2 IP>/wordpress/`**. This will lead you to the WordPress setup wizard where you can finalize the installation process.

> [!NOTE]
Replace **<EC2 IP>** with the IP address of your EC2 instance when accessing your WordPress page.

- Select your preferred language and then click on **Continue** to proceed.

![39](img/Screenshot%20(142).png)

- Enter the required information and click on **Install WordPress** once you have finished.

  - **Site Title:** Enter the name of your WordPress website. It's recommended to use your domain name for better optimization.
  - **Username:** Choose a username for logging into WordPress.
  - **Password:** Set a secure password to protect your WordPress account.
  - **Your email:** Provide your email address to receive updates and notifications.
  - **Search engine visibility:** You can leave this box unchecked to prevent search engines from indexing your site until it's ready.

  ![40](img/Screenshot%20(143).png)

- WordPress has been successfully installed. You can now log in to your admin dashboard using the previously set up information by clicking the **Log In** button.

























![41](img/Screenshot%20(144).png)

- Enter your username and password, then click **Log In** to access your WordPress admin dashboard.

- Once you successfully log in, you will be greeted by the WordPress dashboard page.

![42](img/Screenshot%20(145).png)

---

### Create An A Record

To make your website accessible via your domain name rather than the IP address, you'll need to set up a DNS record. I did this by buying my domain from Namecheap and then moving hosting to AWS Route 53, where I set up an A record.

> [!NOTE]
Visit [**project1**](https://github.com/pinea1111/project-1-docu/blob/master/project1docu.md) for instructions on how to create a hosted zone.

- To update your Apache configuration file in the sites-available directory to point to your domain name, use the command: **`sudo nano /etc/apache2/sites-available/projectlamp.conf`**.

> [!NOTE]
This command opens the **`projectlamp.conf`** file in the nano text editor with superuser privileges **(`sudo`)**. Within the editor, adjust the necessary details to reflect your domain name configuration.

- Ensure that the server settings in your Apache configuration point to your domain name, and that the document root accurately points to your WordPress directory. Once you've made these adjustments, save the changes and exit the editor.

```
<VirtualHost *:80>
    ServerName <Your root domain name>
    ServerAlias <Your sub domain name>
    ServerAdmin webmaster@<Your root domain name>

    DocumentRoot /var/www/html/wordpress

    <Directory /var/www/html/wordpress>
        Options Indexes FollowSymLinks
       # AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
> [!NOTE]
The new configuration defines how Apache should handle requests for your domain, and its subdomain. With this configuration: Apache will handle requests for **nkoloagu.online** and **www.nkoloagu.online**. Files will be served from the **`/var/www/html/wordpress`** directory. Directory listings and symbolic links are allowed. The directory can be accessed by any client.
Error logs will be written to **`/var/log/apache2/error.log`**. Access logs will be written to **`/var/log/apache2/access.log`** in the combined log format.

- To update your **`wp-config.php`** file with DNS settings, use the following command: **`sudo nano wp-config.php`** and add these lines to the file:

```
/** MY DNS SETTINGS */
define('WP_HOME', 'http://<domain name>');

define('WP_SITEURL', 'http://<domain name>');
```

Replace **`http://<domain name>`** with your actual domain name. Save the changes and exit the editor.

- Reload your Apache server to apply the changes with the command: **`sudo systemctl reload apache2`**, After reloading, visit your website at **`http://<domain name>`** to view your WordPress site. Replace **<domain name>** with your actual domain name.

![43](img/Screenshot%20(147).png)

- To log in to your WordPress admin portal, visit **`http://<domain name>/wp-admin`**, Enter your **username** and **password**, then click on **log In**. *Replace **<domain name>** with your actual domain name.*

![44](img/Screenshot%20(148).png)

> [!NOTE]
My domain name is **nkoloagu.online**, so i'll visit **`http://nkoloagu.online/wp-admin`**.

- Now that your WordPress site is successfully configured to use your domain name, the next step is to secure it by requesting an SSL/TLS certificate.

![45](img/Screenshot%20(149).png)

---

### Install certbot and Request For an SSL/TLS Certificate

- Install certbot by executing the following commands:
`sudo apt update`
`sudo apt install certbot python3-certbot-apache`

- Run the command **`sudo certbot --apache`** to request your SSL/TLS certificate. Follow the instructions provided by Certbot to select the domain name for which you want to enable HTTPS.

![46](img/Screenshot%20(150).png)

- You should receive a message confirming that the certificate has been successfully obtained.

![47](img/Screenshot%20(151).png)

- Visit your website to confirm, and you'll notice that the **"not secure"** warning no longer appears, indicating that your site is now secure with HTTPS.

![48](img/Screenshot%20(153).png)

![49](img/Screenshot%20(154).png)

---
---

## End of project 4

