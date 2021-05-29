# DFD4043 - Cloud Computing Part 2
## Stateful webapp deployment with EC2, S3, and Amplify

### Scenario:
> A company is searching for possible web developer for its corporate website. You are
required to provide a website prototype using the AWS cloud environment to a client. Work in a team of three to four people. This time it will be a stateful web application.

# 1. The Hard way with EC2 instance
> We will deploy an EC2 instance as a standalone static website.

1. OS: Amazon Linux 2
2. Stack: LAMP

### Prepare the LAMP Stack
```bash
# Quick Update
sudo yum update -y

#Install the lamp-mariadb10.2-php7.2 and php7.2 Amazon Linux Extras repositories
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2

#Now that your instance is current, you can install the Apache web server, MariaDB, and PHP software packages.
sudo yum install -y httpd mariadb-server
sudo systemctl start httpd && sudo systemctl enable httpd

#Add your user (in this case, ec2-user) to the apache group.
sudo usermod -a -G apache ec2-user

#Change the group ownership of /var/www and its contents to the apache group.
sudo chown -R ec2-user:apache /var/www

#Change the group ownership of /var/www and its contents to the apache group.
sudo chown -R ec2-user:apache /var/www

#To add group write permissions and to set the group ID on future subdirectories, change the directory permissions of /var/www and its subdirectories.
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;

#To add group write permissions, recursively change the file permissions of /var/www and its subdirectories:
find /var/www -type f -exec sudo chmod 0664 {} \;
```

### Test your LAMP server
```bash
Create a PHP file in the Apache document root.
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

#Verify Public DNS
http://my.public.dns.amazonaws.com/phpinfo.php

#You can also verify that all of the required packages were installed with the following command.
sudo yum list installed httpd mariadb-server php-mysqlnd

# Delete the phpinfo.php file. Although this can be useful information, it should not be broadcast to the internet for security reasons.
rm /var/www/html/phpinfo.php
```
### Secure the database server
```bash
# Start the MariaDB server.
sudo systemctl start mariadb

# Run mysql_secure_installation.
sudo mysql_secure_installation

### Install phpMyAdmin
# Install the required dependencies.
sudo yum install php-mbstring php-xml -y

# Restart Apache.
sudo systemctl restart httpd

# Restart php-fpm.
sudo systemctl restart php-fpm

# Navigate to the Apache document root at /var/www/html.
cd /var/www/html

# Down Latest phpmyadmin
https://www.phpmyadmin.net/downloads/

# Create a phpMyAdmin folder and extract the package into it with the following command.
curl -O https://files.phpmyadmin.net/phpMyAdmin/5.1.0/phpMyAdmin-5.1.0-all-languages.tar.gz

mkdir phpMyAdmin && tar -xvzf phpMyAdmin-5.1.0-all-languages.tar.gz -C phpMy
Admin --strip-components 1

# Delete the phpMyAdmin-latest-all-languages.tar.gz tarball.
rm phpMyAdmin-latest-all-languages.tar.gz

# (Optional) If the MySQL server is not running, start it now.
sudo systemctl start mariadb

```
[reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html#prepare-lamp-server)

## 2. Install Wordpress
```bash
# To download and unzip the WordPress installation package
wget https://wordpress.org/latest.tar.gz

## Unzip
tar -xzf latest.tar.gz

## To create a database user and database for your WordPress installation
sudo systemctl start mariadb

# To create and edit the wp-config.php file
mysql -u root -p
CREATE USER 'wordpress-user'@'localhost' IDENTIFIED BY 'p@ssw0rdYangS@ngatKuat';
CREATE DATABASE `wordpress-db`;
GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "wordpress-user"@"localhost";
FLUSH PRIVILEGES;
```

## 3. To create and edit the wp-config.php file
```bash
# Copy wp-config-sample
cp wordpress/wp-config-sample.php wordpress/wp-config.php

# Edit wp-config.php
vim wordpress/wp-config.php

define('DB_NAME', 'wordpress-db');
define('DB_USER', 'wordpress-user');
define('DB_PASSWORD', 'your_strong_password');
```
[Salt and Key](https://api.wordpress.org/secret-key/1.1/salt/)

## 4. To install your WordPress files under the Apache document root

```bash
cp -r wordpress/* /var/www/html/

# Alternative other than document root.
mkdir /var/www/html/blog
cp -r wordpress/* /var/www/html/blog/
```
## 5. To allow WordPress to use permalinks
```bash
sudo vim /etc/httpd/conf/httpd.conf
# with Vim Jump to line 151

#Find the section that starts with <Directory "/var/www/html">.

<Directory "/var/www/html">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    Options Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #
    AllowOverride None

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>

```

## 6. To install the PHP graphics drawing library on Amazon Linux 2
```bash
sudo yum install php-gd
sudo yum list installed | grep php-gd
yum list | grep php-gd
sudo yum install php-gd
```

## 7. To fix file permissions for the Apache web server
```bash
#Grant file ownership of /var/www and its contents to the apache user.
sudo chown -R apache /var/www
#Grant group ownership of /var/www and its contents to the apache group.
sudo chgrp -R apache /var/www
#Change the directory permissions of /var/www and its subdirectories to add group write permissions and to set the group ID on future subdirectories.
sudo chmod 2775 /var/www
find /var/www -type d -exec sudo chmod 2775 {} \;
#Recursively change the file permissions of /var/www and its subdirectories to add group write permissions.
find /var/www -type f -exec sudo chmod 0664 {} \;
#Restart the Apache web server to pick up the new group and permissions.
sudo systemctl restart httpd && sudo service httpd restart
```

## 8. To run the WordPress installation script with Amazon Linux 2
```bash
sudo systemctl enable httpd && sudo systemctl enable mariadb
sudo systemctl status mariadb
sudo systemctl start mariadb
sudo systemctl status httpd
sudo systemctl start httpd
```

## 9. Login credential
```bash
superadmin
LeL3L8E(xIf8CHNAD*
```

# Preferred way with Amazon Lightsail

1. Create Instance
2. Create Database
3. Create IAM Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObjectAcl",
                "s3:GetObject",
                "s3:PutBucketAcl",
                "s3:ListBucket",
                "s3:DeleteObject",
                "s3:GetBucketAcl",
                "s3:GetBucketLocation",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::<your bucket name>",
                "arn:aws:s3:::<your bucket name>/*"
            ]
        }
    ]
}
```
4. Configure managed database

> Load this env variable to the OS
```bash
# Before
LSDB_USERNAME=<DatabaseUserName>
LSDB_ENDPOINT=<DatabaseEndpoint>
LSDB_PASSWORD='<DatabasePassword>'
ACCESS_KEY=<AccessKeyID>
SECRET_KEY=<SecretAccessKey>

# After
LSDB_USERNAME='wordpressUser'
LSDB_ENDPOINT='ls-0b384abc5dd97ae16830de515c951469261f7095.c46fmzhi3xzu.ap-southeast-1.rds.amazonaws.com'
LSDB_PASSWORD='0F!|99Xu%&N8=.Hj^Zi-j-t(}usF)lgG'
ACCESS_KEY='AKIA4UHMPSGHMZGJXRFI'
SECRET_KEY='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'
```
```bash
set -o allexport; source env.txt; set +o allexport
```

5. Dump the database to localhost
```bash
sudo mysqldump -u root --databases bitnami_wordpress --single-transaction --order-by-primary -p > dump.sql
```

6. Dump it to RDS
```bash
cat dump.sql | sudo mysql --user $LSDB_USERNAME --host $LSDB_ENDPOINT -p
```

7. Backup wp-config.php before modify
```bash
cp /home/bitnami/apps/wordpress/htdocs/wp-config.php /home/bitnami/apps/wordpress/htdocs/wp-config.php.bak
```

8. Obtain the WPDB_PASSWORD
```bash
cat /home/bitnami/apps/wordpress/htdocs/wp-config.php | grep DB_PASSWORD | cut -d \' -f 4
```

9. Update The env variable file with the following config
```bash
## Add the WPDB variable and add 3306 port at LSDB_ENDPOINT
LSDB_USERNAME='wordpressUser'
LSDB_ENDPOINT='ls-0b384abc5dd97ae16830de515c951469261f7095.c46fmzhi3xzu.ap-southeast-1.rds.amazonaws.com':3306
LSDB_PASSWORD='0F!|99Xu%&N8=.Hj^Zi-j-t(}usF)lgG'
ACCESS_KEY='AKIA4UHMPSGHMZGJXRFI'
SECRET_KEY='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'
WPDB_PASSWORD='8acb9c7bde'

## Load the env variable
set -o allexport; source env.txt; set +o allexport
```
10. Invoke wordpress cli to add the MySQL managed database user name, password, and endpoint to the wp-config.php file

```bash
wp config set DB_USER $LSDB_USERNAME && wp config set DB_PASSWORD $LSDB_PASSWORD && wp config set DB_HOST $LSDB_ENDPOINT
```
11. Create IAM credential file
```bash
cat <<EOT >> credfile.txt
define( 'AS3CF_SETTINGS', serialize( array (
    'provider' => 'aws', 
    'access-key-id' => '$ACCESS_KEY', 
    'secret-access-key' => '$SECRET_KEY',
) ) );
EOT
```
12. Enter the following command to insert the contents of the credfile.txt into the wp-config.php file.
```bash
sed -i "/define( 'WP_DEBUG', false );/r credfile.txt" /home/bitnami/apps/wordpress/htdocs/wp-config.php
```

13. Delete the credential file
```bash
sudo rm –r credfile.txt
```

14. Enter the following command to correct the permissions for the wp-config.php file, and the backup that you created.
```bash
sudo chown bitnami:daemon /home/bitnami/apps/wordpress/htdocs/wp-config.php && sudo chown bitnami:daemon /home/bitnami/apps/wordpress/htdocs/wp-config.php.bak
```
15. Restart Service
```bash
sudo /opt/bitnami/ctlscript.sh restart
```
