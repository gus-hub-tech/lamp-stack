# lamp-stack
How To Install LAMP Stack (Linux, Apache, MySQL, PHP) 


## Overview

This guide provides complete instructions for setting up a LAMP (Linux, Apache, MySQL/MariaDB, PHP) stack on Amazon Linux 2023. The LAMP stack is a popular open-source web development platform used to host dynamic websites and applications.

### What is LAMP?

- **L**inux: Operating system foundation (Amazon Linux 2023)
- **A**pache: Web server for handling HTTP requests
- **M**ySQL/MariaDB: Relational database management system
- **P**HP: Server-side scripting language

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                          AWS Cloud                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    VPC (Virtual Private Cloud)           │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Availability Zone                      │  │  │
│  │  │                                                     │  │  │
│  │  │  ┌───────────────────────────────────────────────┐  │  │  │
│  │  │  │            EC2 Instance (t3.micro)           │  │  │  │
│  │  │  │         Amazon Linux 2023.9.20251208         │  │  │  │
│  │  │  │                                               │  │  │  │
│  │  │  │  ┌─────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │              Web Layer                 │  │  │  │  │
│  │  │  │  │                                         │  │  │  │  │
│  │  │  │  │  ┌─────────────────────────────────┐    │  │  │  │  │
│  │  │  │  │  │         Apache HTTP Server      │    │  │  │  │  │
│  │  │  │  │  │            (httpd 2.4.x)        │    │  │  │  │  │
│  │  │  │  │  │                                 │    │  │  │  │  │
│  │  │  │  │  │  • Virtual Hosts               │    │  │  │  │  │
│  │  │  │  │  │  • SSL/TLS Support             │    │  │  │  │  │
│  │  │  │  │  │  • .htaccess Support           │    │  │  │  │  │
│  │  │  │  │  └─────────────────────────────────┘    │  │  │  │  │
│  │  │  │  └─────────────────────────────────────────┘  │  │  │  │
│  │  │  │                                               │  │  │  │
│  │  │  │  ┌─────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │           Application Layer             │  │  │  │  │
│  │  │  │  │                                         │  │  │  │  │
│  │  │  │  │  ┌─────────────────────────────────┐    │  │  │  │  │
│  │  │  │  │  │            PHP 8.2.x           │    │  │  │  │  │
│  │  │  │  │  │                                 │    │  │  │  │  │
│  │  │  │  │  │  • php-mysqlnd                 │    │  │  │  │  │
│  │  │  │  │  │  • php-fpm                     │    │  │  │  │  │
│  │  │  │  │  │  • php-curl, php-gd, php-xml   │    │  │  │  │  │
│  │  │  │  │  │  • php-opcache (performance)    │    │  │  │  │  │
│  │  │  │  │  └─────────────────────────────────┘    │  │  │  │  │
│  │  │  │  └─────────────────────────────────────────┘  │  │  │  │
│  │  │  │                                               │  │  │  │
│  │  │  │  ┌─────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │            Database Layer               │  │  │  │  │
│  │  │  │  │                                         │  │  │  │  │
│  │  │  │  │  ┌─────────────────────────────────┐    │  │  │  │  │
│  │  │  │  │  │        MariaDB 10.5.x          │    │  │  │  │  │
│  │  │  │  │  │      (MySQL Compatible)         │    │  │  │  │  │
│  │  │  │  │  │                                 │    │  │  │  │  │
│  │  │  │  │  │  • Secure Installation         │    │  │  │  │  │
│  │  │  │  │  │  • User Management             │    │  │  │  │  │
│  │  │  │  │  │  • Database Storage            │    │  │  │  │  │
│  │  │  │  │  └─────────────────────────────────┘    │  │  │  │  │
│  │  │  │  └─────────────────────────────────────────┘  │  │  │  │
│  │  │  │                                               │  │  │  │
│  │  │  │  ┌─────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │            File System                  │  │  │  │  │
│  │  │  │  │                                         │  │  │  │  │
│  │  │  │  │  • /var/www/html/ (Web Root)            │  │  │  │  │
│  │  │  │  │  • /etc/httpd/ (Apache Config)          │  │  │  │  │
│  │  │  │  │  • /var/log/httpd/ (Apache Logs)        │  │  │  │  │
│  │  │  │  │  • /etc/my.cnf (MariaDB Config)         │  │  │  │  │
│  │  │  │  └─────────────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      Security Groups                           │
├─────────────────────────────────────────────────────────────────┤
│  Port 22  (SSH)    │ Your IP Address                          │
│  Port 80  (HTTP)   │ 0.0.0.0/0 (Internet)                    │
│  Port 443 (HTTPS)  │ 0.0.0.0/0 (Internet)                    │
│  Port 3306 (MySQL) │ Restricted IPs (Optional)                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Data Flow                               │
└─────────────────────────────────────────────────────────────────┘

    Internet Users
         │
         ▼
   ┌─────────────┐    HTTP/HTTPS     ┌─────────────────┐
   │   Browser   │ ◄──────────────── │  Apache Server  │
   └─────────────┘    (Port 80/443)  │    (httpd)      │
         │                           └─────────────────┘
         │                                     │
         ▼                                     ▼
   Static Content                        ┌─────────────────┐
   (HTML, CSS, JS)                       │   PHP Engine    │
                                         │   (php-fpm)     │
                                         └─────────────────┘
                                                   │
                                                   ▼
                                         ┌─────────────────┐
                                         │   MariaDB       │
                                         │   Database      │
                                         │  (Port 3306)    │
                                         └─────────────────┘
```

### Key Components:

- **AWS EC2 Instance**: Hosts the entire LAMP stack on Amazon Linux 2023
- **Apache HTTP Server**: Handles web requests and serves static/dynamic content
- **PHP Engine**: Processes server-side scripts and connects to database
- **MariaDB Database**: Stores application data with MySQL compatibility
- **Security Groups**: AWS firewall rules controlling network access
- **File System**: Organized directory structure for web files and configurations

## System Information

This guide is specifically designed for:
- **Operating System**: Amazon Linux 2023.9.20251208
- **Kernel**: Linux 6.1.158-180.294.amzn2023.x86_64
- **Architecture**: x86-64
- **Platform**: AWS EC2
- **Instance Type**: t3.micro (or similar)
- **Virtualization**: Amazon (Nitro System)

## Prerequisites

Before starting, ensure you have:
- Amazon Linux 2023 EC2 instance
- Non-root user with sudo privileges (ec2-user by default)
- SSH access to your instance
- AWS Security Groups configured for HTTP/HTTPS traffic
- Basic command line knowledge

### Important Note
Amazon Linux uses **`dnf`** package manager (not `apt` like Ubuntu). Commands in this guide reflect Amazon Linux syntax.

## Installation Steps

### Step 1: Update System

Update all system packages:

```bash
sudo dnf update -y
```

### Step 2: Install Apache (httpd)

Install and start Apache web server:

```bash
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```
<img width="947" height="370" alt="httpd" src="https://github.com/user-attachments/assets/d58293af-c0aa-4551-8d3f-287d23f18daa" />

#### Configure AWS Security Group

Make sure your EC2 Security Group allows inbound traffic:
- **Port 80** (HTTP) - from 0.0.0.0/0
- **Port 443** (HTTPS) - from 0.0.0.0/0
- **Port 22** (SSH) - from your IP

<img width="1375" height="637" alt="security" src="https://github.com/user-attachments/assets/0b29e256-a542-42b4-ba2a-a274b07bec3c" />


#### Configure Firewall (Optional)

If using firewalld:

```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

#### Verify Installation

Visit your EC2 public IP in a browser:
```
http://your_ec2_public_ip
```

You should see the Apache test page.

<img width="957" height="272" alt="test" src="https://github.com/user-attachments/assets/3f1788f6-c3db-4f06-a066-4ace71595c97" />


#### Find Your EC2 Public IP

```bash
# From AWS metadata service
curl http://169.254.169.254/latest/meta-data/public-ipv4

# Or from external service
curl http://icanhazip.com

# Or check AWS Console
# EC2 Dashboard → Instances → Select your instance → Public IPv4 address
```

### Step 3: Install MariaDB (MySQL Alternative)

Amazon Linux 2023 uses MariaDB as the MySQL-compatible database:

```bash
sudo dnf install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```
<img width="960" height="376" alt="mariadb" src="https://github.com/user-attachments/assets/653074ad-7331-441a-99b2-376ed1c9fce1" />


#### Secure MariaDB Installation

Run the security script:

```bash
sudo mysql_secure_installation
```

Follow the prompts:
1. **Enter current password**: Press Enter (no password set initially)
2. **Switch to unix_socket authentication**: Y
3. **Change root password**: Y, then enter a strong password
4. **Remove anonymous users**: Y
5. **Disallow root login remotely**: Y
6. **Remove test database**: Y
7. **Reload privilege tables**: Y

#### Test MariaDB Access

```bash
sudo mysql -u root -p
```

Enter your password and you should see:

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 10.5.x-MariaDB

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Exit with:
```sql
exit
```
<img width="952" height="275" alt="mariadb-access" src="https://github.com/user-attachments/assets/8d54758c-c6aa-46df-af03-a7743bc76276" />


### Step 4: Install PHP

Install PHP and required modules:

```bash
sudo dnf install php php-mysqlnd php-fpm php-cli php-xml php-mbstring php-json php-gd php-curl -y
```

Verify installation:

```bash
php -v
```

Expected output:
```
PHP 8.2.x (cli) (built: ...)
```
<img width="952" height="667" alt="php" src="https://github.com/user-attachments/assets/ccd74719-3390-414c-acf2-843eeb71d9f3" />

#### Restart Apache

After installing PHP:

```bash
sudo systemctl restart httpd
sudo systemctl status httpd
```

#### Install Additional PHP Modules (Optional)

Search for available PHP modules:

```bash
dnf search php-
```

Install additional modules as needed:

```bash
sudo dnf install php-opcache php-zip php-intl -y
```

**Common module descriptions:**
- `php-mysqlnd`: MySQL Native Driver for PHP
- `php-cli`: Command-line PHP scripting
- `php-fpm`: FastCGI Process Manager
- `php-curl`: HTTP request support for APIs
- `php-mbstring`: Multibyte string handling (UTF-8)
- `php-xml`: XML parsing capabilities
- `php-json`: JSON handling
- `php-gd`: Image processing library
- `php-zip`: ZIP archive handling
- `php-opcache`: Performance optimization

### Step 5: Create Virtual Host

Create a directory for your website:

```bash
sudo mkdir -p /var/www/your_domain
sudo chown -R $USER:$USER /var/www/your_domain
sudo chmod -R 755 /var/www
```

Create virtual host configuration:

```bash
sudo nano /etc/httpd/conf.d/your_domain.conf
```

Add this configuration:

```apache
<VirtualHost *:80>
    ServerName your_domain
    ServerAlias www.your_domain
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/your_domain
    
    <Directory /var/www/your_domain>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog /var/log/httpd/your_domain_error.log
    CustomLog /var/log/httpd/your_domain_access.log combined
</VirtualHost>
```

**Note**: For testing without a domain, you can use your EC2 public IP directly with the default `/var/www/html` directory.

Test configuration and restart Apache:

```bash
sudo httpd -t
sudo systemctl restart httpd
```

Create a test page:

```bash
nano /var/www/your_domain/index.html
```

```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>lamp_stack website</title>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #1e3c72, #2a5298);
      color: #fff;
      text-align: center;
      display: flex;
      flex-direction: column;
      justify-content: center;
      height: 100vh;
      overflow: hidden;
    }

    h1 {
      font-size: 3rem;
      margin-bottom: 0.5rem;
      animation: fadeInDown 2s ease-in-out;
    }

    p {
      font-size: 1.5rem;
      animation: fadeInUp 2s ease-in-out;
</body>ello World from Amazon Linux 2023!</h1> 40px #00ffff; } #00e6e6;
</html>is is the landing page of my <strong>lamp-stack</strong>.</p>
<b<div class="glow">🚀 Powered by LAMP & AWS</div>#00e6e6; }

```

For quick testing with default directory:

```bash
echo "<h1>Hello from Amazon Linux 2023</h1>" | sudo tee /var/www/html/index.html
```

https://github.com/user-attachments/assets/703e2627-931f-4e6a-a7cd-48a468700aae


### Step 6: Test PHP Processing

Create a PHP info file:

```bash
sudo nano /var/www/html/info.php
```

```php
<?php
phpinfo();
?>
```

Access it in your browser:
```
http://your_ec2_public_ip/info.php
```
<img width="957" height="610" alt="php-info" src="https://github.com/user-attachments/assets/6edcec96-0727-406b-9cc9-fa1e44fe8d9c" />


**Important:** Delete this file after testing as it exposes sensitive information:
```bash
sudo rm /var/www/html/info.php
```

### Step 7: Test Database Connection (Optional)

#### Create Test Database

Log into MariaDB:

```bash
sudo mysql -u root -p
```

Create database and user:

```sql
CREATE DATABASE example_database;
CREATE USER 'example_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON example_database.* TO 'example_user'@'localhost';
FLUSH PRIVILEGES;
exit
```

<img width="910" height="261" alt="data-base" src="https://github.com/user-attachments/assets/736f8727-5550-478b-80e1-08ffdbe6e2ee" />

Test user login:

```bash
mysql -u example_user -p
```

Create test table:

```sql
USE example_database;

CREATE TABLE example_database.todo_list (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);

INSERT INTO todo_list (content) VALUES ("My first important item");
INSERT INTO todo_list (content) VALUES ("My second important item");
INSERT INTO todo_list (content) VALUES ("My third important item");

SELECT * FROM todo_list;
exit
```

<img width="952" height="312" alt="test-table" src="https://github.com/user-attachments/assets/a0edc07a-ac5e-4b12-b12b-42064e7ebc7c" />


#### Create PHP Database Test Script

The script will connect to MySQL and query for your content. 

```bash
sudo nano /var/www/html/todo_list.php
```

```php
<?php
$user = "example_user";
$password = "secure_password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO List from MariaDB</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

Access in browser:
```
http://your_ec2_public_ip/todo_list.php
```

<img width="952" height="373" alt="todo" src="https://github.com/user-attachments/assets/5b01a3dd-33a7-4ba0-b107-ea92582aa6a1" />


## Advanced Configuration [Optional]

### Remote MariaDB Access

Edit MariaDB configuration:

```bash
sudo nano /etc/my.cnf.d/mariadb-server.cnf
```

Find and change the bind-address (or add if not present):
```ini
[mysqld]
bind-address = 0.0.0.0
```

Restart MariaDB:
```bash
sudo systemctl restart mariadb
```

Create remote user:

```sql
sudo mysql -u root -p
CREATE USER 'remote_user'@'%' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
```

Configure AWS Security Group:
- Add inbound rule for **Port 3306** (MySQL/Aurora) from specific IP or IP range

Configure firewalld (if using):
```bash
sudo firewall-cmd --permanent --add-port=3306/tcp
sudo firewall-cmd --reload
```

**Security Warning**: For production, use VPN or SSH tunneling instead of exposing port 3306.

### Enable .htaccess Support

Edit Apache configuration:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Find the `<Directory "/var/www/html">` section and change `AllowOverride None` to:
```apache
AllowOverride All
```

Restart Apache:
```bash
sudo systemctl restart httpd
```

## Security Best Practices

1. **Enable HTTPS** with Let's Encrypt SSL certificate
2. Run `mysql_secure_installation` after MariaDB setup
3. Use strong passwords for all database users
4. Configure AWS Security Groups to restrict access:
   - Allow SSH (22) only from your IP
   - Allow HTTP (80) from anywhere
   - Allow HTTPS (443) from anywhere
   - Restrict MySQL (3306) to specific IPs only
5. Keep all packages updated regularly:
   ```bash
   sudo dnf update -y
   ```
6. Disable directory listing in Apache
7. Configure SELinux policies appropriately
8. Use AWS IAM roles for EC2 instead of hardcoded credentials
9. Enable AWS CloudWatch for monitoring
10. Regular backups of databases and web files

### SELinux Configuration

Amazon Linux 2023 has SELinux enabled by default. If you encounter permission issues:

```bash
# Check SELinux status
getenforce

# Allow Apache to connect to network (for remote DB connections)
sudo setsebool -P httpd_can_network_connect 1

# Allow Apache to read/write in web directories
sudo chcon -R -t httpd_sys_rw_content_t /var/www/html/
```

## Troubleshooting

### Apache not starting
```bash
sudo systemctl status httpd
sudo journalctl -xeu httpd
sudo httpd -t  # Test configuration
```

Common issues:
- Check if port 80/443 is already in use: `sudo netstat -tulpn | grep :80`
- Verify configuration syntax: `sudo httpd -t`
- Check SELinux: `sudo ausearch -m avc -ts recent`

### MariaDB access denied
Reset root password:
```bash
sudo systemctl stop mariadb
sudo mysqld_safe --skip-grant-tables &
mysql -u root
```

```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
exit
```

```bash
sudo systemctl restart mariadb
```

### PHP not processing
Verify PHP module is loaded:
```bash
php -m  # List all loaded modules
sudo systemctl restart httpd
```

Check Apache error logs:
```bash
sudo tail -f /var/log/httpd/error_log
```

### Permission Denied Errors

If you encounter permission issues:

```bash
# Fix ownership
sudo chown -R apache:apache /var/www/html/

# Fix permissions
sudo chmod -R 755 /var/www/html/

# Check SELinux
sudo setsebool -P httpd_unified 1
```

### Database Connection Issues

```bash
# Check MariaDB is running
sudo systemctl status mariadb

# Test connection
mysql -u username -p -h localhost

# Check MariaDB logs
sudo tail -f /var/log/mariadb/mariadb.log
```

### View Service Logs

```bash
# Apache logs
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log

# MariaDB logs
sudo tail -f /var/log/mariadb/mariadb.log

# System logs
sudo journalctl -xe
```

## Useful Commands

### Service Management
```bash
# Start services
sudo systemctl start httpd
sudo systemctl start mariadb

# Stop services
sudo systemctl stop httpd
sudo systemctl stop mariadb

# Restart services
sudo systemctl restart httpd
sudo systemctl restart mariadb

# Enable on boot
sudo systemctl enable httpd
sudo systemctl enable mariadb

# Check status
sudo systemctl status httpd
sudo systemctl status mariadb
```

### File Locations
```bash
# Apache configuration
/etc/httpd/conf/httpd.conf
/etc/httpd/conf.d/

# Virtual host configurations
/etc/httpd/conf.d/your_domain.conf

# Web root
/var/www/html/

# Apache logs
/var/log/httpd/access_log
/var/log/httpd/error_log

# MariaDB configuration
/etc/my.cnf
/etc/my.cnf.d/mariadb-server.cnf

# MariaDB logs
/var/log/mariadb/mariadb.log

# PHP configuration
/etc/php.ini
/etc/php.d/
```

### Package Management
```bash
# Update all packages
sudo dnf update -y

# Search for packages
dnf search package_name

# Install package
sudo dnf install package_name -y

# Remove package
sudo dnf remove package_name -y

# List installed packages
dnf list installed | grep php
dnf list installed | grep httpd
dnf list installed | grep mariadb

# Check for available updates
dnf check-update
```

## Next Steps

### Essential Security
- **Install SSL Certificate** with Let's Encrypt/Certbot
  ```bash
  sudo dnf install certbot python3-certbot-apache -y
  sudo certbot --apache -d your_domain.com
  ```
- **Configure AWS Security Groups** properly
- **Set up AWS CloudWatch** for monitoring
- **Enable AWS Backup** for EC2 snapshots

### Enhance Your Setup
- **Install phpMyAdmin** for database management
  ```bash
  sudo dnf install phpMyAdmin -y
  ```
- **Configure multiple virtual hosts** for different websites
- **Set up automated backups** using AWS Backup or scripts
- **Implement caching** (Redis/Memcached) for better performance
- **Configure log rotation** to manage disk space

### Performance Optimization
- Enable PHP OPcache
- Configure Apache MPM (Multi-Processing Modules)
- Set up Content Delivery Network (CloudFront)
- Implement database query optimization
- Use AWS Elastic Load Balancer for scaling

### Monitoring
- Set up AWS CloudWatch alarms
- Configure log monitoring
- Install monitoring tools (htop, iotop, nethogs)
  ```bash
  sudo dnf install htop iotop nethogs -y
  ```
  ## Common Use Cases

Perfect for hosting:
- **WordPress** websites and blogs
- **Laravel, Symfony, CodeIgniter** applications
- **Custom PHP web portals** and dashboards
- **RESTful API servers** with PHP backend
- **E-commerce platforms** (Magento, OpenCart, WooCommerce)
- **Content Management Systems** (Drupal, Joomla)
- **Web applications** with dynamic content

## Version Information

This guide is specifically designed for:
- **Operating System**: Amazon Linux 2023.9.20251208
- **Kernel**: Linux 6.1.158-180.294.amzn2023.x86_64
- **Apache**: 2.4.x (httpd)
- **MariaDB**: 10.5.x (MySQL-compatible)
- **PHP**: 8.2.x
- **Platform**: AWS EC2 (t3.micro or similar)
- **Architecture**: x86-64

## AWS-Specific Considerations

### Cost Optimization
- Use **t3.micro** for development (Free Tier eligible)
- Implement **Auto Scaling** for production
- Use **Reserved Instances** for long-term projects
- Configure **CloudWatch** billing alarms

### High Availability
- Deploy across multiple **Availability Zones**
- Use **Elastic Load Balancer (ELB)**
- Implement **RDS for MariaDB** instead of self-hosted
- Configure **Route 53** for DNS failover

### Backup Strategy
- Enable **AWS Backup** for EC2 snapshots
- Use **RDS automated backups** if using RDS
- Store backups in **S3** with lifecycle policies
- Test restore procedures regularly

## Additional Resources

### Documentation
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Amazon Linux 2023 Documentation](https://docs.aws.amazon.com/linux/al2023/)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/2.4/)
- [MariaDB Documentation](https://mariadb.com/kb/en/documentation/)
- [PHP Documentation](https://www.php.net/docs.php)

### AWS Services
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) - Free SSL certificates
- [Amazon RDS](https://aws.amazon.com/rds/) - Managed database service
- [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) - Automated deployment

### Community Support
- [AWS re:Post](https://repost.aws/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/amazon-linux-2023)
- [Amazon Linux Forums](https://forums.aws.amazon.com/forum.jspa?forumID=228)

## License

This documentation is created for educational purposes and based on official Amazon Linux and Apache documentation.

---

**Last Updated**: December 2024  
**Compatible with**: Amazon Linux 2023.9.20251208  
**Tested on**: AWS EC2 t3.micro instance# LAMP Stack Installation Guide for Amazon Linux 2023
