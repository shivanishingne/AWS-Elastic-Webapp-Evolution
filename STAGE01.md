## STAGE 1: Setting up and Building Wordpress manually

---

## What was achieved:
1.  Setup the environment from which Wordpress will run.
2.  Configured SSM Parameter Store to hold imp. configuration info.
3.  Performed manual installation of Wordpress and a database on the same EC2 instance.

---

### STAGE 1A: Logging into AWS 
- Configured the VPC and subnets for the architecture
- Configured Security Group
- Created IAM role with the required permissions

### STAGE 1B: Create an EC2 Instance to run Wordpress
- AMI selected was: `Amazon Linux 2 AMI (HVM), SSD Volume Type` ;  `64-bit (x86)` 

### STAGE 1C: Create SSM Parameter Store values for Wordpress
- Storing configuration information within the SSM Parameter store scales much better than attempting to script them in some way.
- Created Parameters for the following:
    - DBName
    - DBEndpoint
    - DBUser
    - DBPassword
    - DBRootPassword

### STAGE 1D: Connecting to the instance using EC2 instance connect, and installing a database and Wordpress

#### Bringing in all the parameter values from SSM
Eg:
```
YourDBUser=$(aws ssm get-parameters --region <ENTER_YOUR_REGION> --names <ENTER_PARAMETER_NAME> --query Parameters[0].Value)

YourDBUser=`echo $YourDBUser | sed -e 's/^"//' -e 's/"$//'`
```

#### Installing updates
```
sudo yum -y update
sudo yum -y upgrade
```

#### Installing pre-req and web server
```
sudo yum install -y mariadb-server httpd wget
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
```

#### Setting up DB and HTTP server to running and start by default
```
sudo systemctl enable httpd
sudo systemctl enable mariadb
sudo systemctl start httpd
sudo systemctl start mariadb
```

#### Setting up MariaDB Root password
```
sudo mysqladmin -u root password $<YOUR_PARAMETER_ROOTPASSWORD_NAME>
```

#### Downloading and extracting Wordpress
```
sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
cd /var/www/html
sudo tar -zxvf latest.tar.gz
sudo cp -rvf wordpress/* .
sudo rm -R wordpress
sudo rm latest.tar.gz
```

#### Configuring the wordpress wp-config.php file
```
sudo cp ./wp-config-sample.php ./wp-config.php
sudo sed -i "s/'database_name_here'/'$<YOUR_PARAMETER_DBNAME>'/g" wp-config.php
sudo sed -i "s/'username_here'/'$<YOUR_PARAMETER_DBUSER>'/g" wp-config.php
sudo sed -i "s/'password_here'/'$<YOUR_PARAMETER_DBPASSWORD>'/g" wp-config.php
```

#### Fixing Permissions on the filesystem
```
sudo usermod -a -G apache ec2-user   
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www
sudo find /var/www -type d -exec chmod 2775 {} \;
sudo find /var/www -type f -exec chmod 0664 {} \;
```

#### Creating Wordpress User, set its password, create the database and configure permissions
```
sudo echo "CREATE DATABASE $<YOUR_PARAMETER_DBNAME>;" >> /tmp/db.setup

sudo echo "CREATE USER '$<YOUR_PARAMETER_DBNAME>'@'localhost' IDENTIFIED BY '$<YOUR_PARAMETER_DBPASSWORD>';" >> /tmp/db.setup

sudo echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup

sudo echo "FLUSH PRIVILEGES;" >> /tmp/db.setup

sudo mysql -u root --password=$<YOUR_PARAMETER_DBROOTPASSWORD> < /tmp/db.setup

sudo rm /tmp/db.setup
```

#### Testing whether wordpress is installed
  
- Select the EC2 instance and copy-paste the public IPv4 IP in a new tab.

#### Performing initial configuration and making a post
- Publish a post by following the on screen instructions.

---


## Limitations with this configuration:

So far the limitations faced are:

- No automation of database and application (manually built)
- The database and application are on the same instance, neither can scale without the other
- The database of the application is on an instance; scaling IN/OUT risks this media
- The application media and UI store is local to an instance; scaling IN/OUT risks this media
- Customer Connections are directly to an instance; no health-checks or auto-healing
