# Before You Install

## Setup Passwordless SSH
  1. run ssh-keygen -t rsa 
  2. run ssh-copy-id -i ~/.ssh/id_rsa.pub remote-host 
  3. set permissions on .ssh directory and authorized_keys file
    3.1 run chmod 700 .ssh 
  
## Configure Network Name
  1. Set the hostname to a unique name (not localhost)
    1.1 sudo hostnamectl set-hostname foo-1.example.com
  2. Edit /etc/hosts with the IP address and fully qualified domain name (FQDN) of each host in the cluster
    2.1 run vi /etc/hosts
        1.1.1.1  foo-1.example.com  foo-1
        2.2.2.2  foo-2.example.com  foo-2
        3.3.3.3  foo-3.example.com  foo-3
        4.4.4.4  foo-4.example.com  foo-4
  3. Edit /etc/sysconfig/network with the FQDN of this host only
    3.1 " HOSTNAME=foo-1.example.com "
  4. Verify that each host consistently identifies to the network
    4.1 Run uname -a and check that the hostname matches the output of the hostname command
    4.2 Run host -v -t A $(hostname) and verify that the output matches the hostname command.
        The IP address should be the same as reported by ifconfig for eth0 (or bond0)
   
## Disabling the Firewall
  1. For iptables, save the existing rule set
      sudo iptables-save > ~/firewall.rules
  2. Disable the firewall
    2.1 RHEL 7 compatible
        sudo systemctl disable firewalld
        sudo systemctl stop firewalld
    2.2 SLES
        sudo chkconfig SuSEfirewall2_setup off
        sudo chkconfig SuSEfirewall2_init off
        sudo rcSuSEfirewall2 stop
    2.3 Ubuntu
        sudo service ufw stop
 
 ## Setting SELinux mode
 
    Security-Enhanced Linux (SELinux) allows you to set access control through policies. If you are having trouble deploying    CDH with your policies, set SELinux in permissive mode on each host before you deploy CDH on your cluster. To set the SELinux mode, perform the following steps on each host.
    
    1. Check the SELinux state
      1.1 getenforce
    2. If the output is either Permissive or Disabled, you can skip this task and continue on to Disabling the Firewall. If          the output is enforcing, continue to the next step.
    3. Open the /etc/selinux/config file (in some systems, the /etc/sysconfig/selinux file)
    4. Change the line SELINUX=enforcing to SELINUX=permissive
    5. Save and close the file
    6. Restart your system or run the following command to disable SELinux immediately
      6.1 setenforce 0
      
 ## Enable an NTP Service
    
    CDH requires that you configure a Network Time Protocol (NTP) service on each machine in your cluster. Most operating systems include the ntpd service for time synchronization. RHEL 7 compatible operating systems use chronyd by default instead of ntpd. If chronyd is running (on any OS), Cloudera Manager uses it to determine whether the host clock is synchronized. Otherwise, Cloudera Manager uses ntpd. To use ntpd for time synchronization:
    1. Install the ntp package:
      1.1 RHEL compatible:
          yum install ntp
      1.2 SLES:
          zypper install ntp
      1.3 Ubuntu:
          apt-get install ntp
    2. Edit the /etc/ntp.conf file to add NTP servers, as in the following example.
        server 0.pool.ntp.org
        server 1.pool.ntp.org
        server 2.pool.ntp.org
    3. Start the ntpd service:
      3.1 RHEL 7 Compatible:
          sudo systemctl start ntpd
      3.2 RHEL 6 Compatible, SLES, Ubuntu:
          sudo service ntpd start
    4. Configure the ntpd service to run at boot:
      4.1 RHEL 7 Compatible:
          sudo systemctl enable ntpd
      4.2 RHEL 6 Compatible, SLES, Ubuntu:
          chkconfig ntpd on
    5. Synchronize the system clock to the NTP server:
        ntpdate -u <ntp_server>
    6. Synchronize the hardware clock to the system clock:
        hwclock --systohc
        
 ## (RHEL 6 Compatible Only) Install Python 2.7 on Hue Hosts 
 
    Hue in CDH 6 requires Python 2.7, which is included by default in RHEL 7 compatible operating systems (OSes). RHEL 6 compatible OSes include Python 2.6. You must install Python 2.7 on all Hue hosts before installing or upgrading to Cloudera Enterprise 6:

    RHEL 6
    1. Make sure that you have access to the Software Collections Library. For more information, see the Red Hat knowledge            base article, How to use Red Hat Software Collections (RHSCL) or Red Hat Developer Toolset (DTS)?.
    2. Install Python 2.7:
       sudo yum install python27
    3. Verify that Python 2.7 is installed:
       source /opt/rh/python27/enable
       python --version
    CentOS 6
    1. Enable the Software Collections Library:
       sudo yum install centos-release-scl
    2. Install the Software Collections utilities:
       sudo yum install scl-utils
    3. Install Python 2.7:
       sudo yum install python27
    4. Verify that Python 2.7 is installed:
       source /opt/rh/python27/enable
       python --version
    Oracle Linux 6
    1. Download the Software Collections Library repository:
       sudo wget -O /etc/yum.repos.d/public-yum-ol6.repo http://yum.oracle.com/public-yum-ol6.repo
    2. Edit /etc/yum.repos.d/public-yum-ol6.repo and make sure that enabled is set to 1, as follows:
       [ol6_software_collections]
       name=Software Collection Library release 3.0 packages for Oracle Linux 6 (x86_64)
       baseurl=http://yum.oracle.com/repo/OracleLinux/OL6/SoftwareCollections/x86_64/
       gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
       gpgcheck=1
       enabled=1
    3. Install the Software Collections utilities:
       sudo yum install scl-utils
    4. Install Python 2.7:
       sudo yum install python27
    5. Verify that Python 2.7 is installed:
       source /opt/rh/python27/enable
       python --version

# Custom Installation Solutions

## Download the parcels or rpm that needed for installation 
    
    CDH 6.3.2
    1. sudo mkdir -p /var/www/html/cloudera-repos
    2. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cdh6/6.3.2/parcels/ -P /var/www/html/cloudera-repos
   
    CM 6.3.1
    1. sudo mkdir -p /var/www/html/cloudera-repos/cm6
    2. wget https://archive.cloudera.com/cm6/6.3.1/repo-as-tarball/cm6.3.1-redhat7.tar.gz
    3. tar xvfz cm6.3.1-redhat7.tar.gz -C /var/www/html/cloudera-repos/cm6 --strip-components=1
    4. chmod -R ugo+rX /var/www/html/cloudera-repos/cm6
    
    GPLExtras6
    1. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/gplextras6/6.3.2/parcels/ -P /var/www/html/cloudera-repos
    
    Accumulo-c5
    1. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/accumulo-c5/parcels/1.7.2/ -P /var/www/html/cloudera-repos

    Spark 2
    1. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/spark2/parcels/2.3.0.cloudera3/ -P /var/www/html/cloudera-repos

    Sqoop Connectors
    1. sudo wget --recursive --no-parent --no-host-directories http://archive.cloudera.com/sqoop-connectors/parcels/latest/ -P /var/www/html/cloudera-repos
    2. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/sqoop-teradata-connector1/latest/ -P /var/www/html/cloudera-repos
    3. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/sqoop-netezza-connector1/latest/ -P /var/www/html/cloudera-repos

    
    Accumulo 6
    1. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/accumulo6/6.1/parcels/ -P /var/www/html/cloudera-repos

    Kafka
    1. sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/kafka/parcels/latest/ -P /var/www/html/cloudera-repos

    Kudu 
    1. sudo wget --recursive --no-parent --no-host-directories http://archive.cloudera.com/kudu/parcels/latest/ -P /var/www/html/cloudera-repos

    CSA ( Flink )
    1. sudo wget --recursive --no-parent --no-host-directories https://xxx:yyyy@archive.cloudera.com/p/csa/1.1.0.0/parcels/ -P /var/www/html/cloudera-repos ( need license )
    
    CFM Nifi
    1. sudo wget --recursive --no-parent --no-host-directories http://archive.cloudera.com/CFM/parcels/1.0.1.0/ -P /var/www/html/cloudera-repos

    sudo chmod -R ugo+rX /var/www/html/cloudera-repos

## Create Local Repo
    1. touch /etc/yum.repos.d/cloudera-repo.repo
    2. vim /etc/yum.repos.d/cloudera-repo.repo 
       [cloudera-repo]
       name=cloudera-repo
       baseurl=http://hostname:8000/html/cloudera-repos/cm/version/
       enabled=1
       gpgcheck=0
    3. change to the directory where all of the sources and parcels is downloaded
       cd /var/www/html/
    4. python -m SimpleHTTPServer 8000
    5. put repos file into all of host server
       scp cloudera-repo.repo hostname2:/etc/yum.repos.d/
       scp cloudera-repo.repo hostname3:/etc/yum.repos.d/
 
 # MariaDB Installation
 
 ## Install MariaDB
    
      1. RHEL compatible
         sudo yum install mariadb-server
      2. SLES
         sudo zypper install mariadb-server
      3. Ubuntu
         sudo apt-get install mariadb-server
 
 ## Configuring and Starting the MariaDB Server
      
      1. Stop the MariaDB server if it is running 
        1.1 RHEL 7 compatible:
            sudo systemctl stop mariadb
        1.2 RHEL 6 compatible, Ubuntu, SLES:
            sudo service mariadb stop
      2. If they exist, move old InnoDB log files /var/lib/mysql/ib_logfile0 and /var/lib/mysql/ib_logfile1 out of                      /var/lib/mysql/ to a backup location.
      3. Determine the location of the option file, my.cnf (/etc/my.cnf by default).
      4. Update my.cnf so that it conforms to the following requirements:
        4.1 To prevent deadlocks, set the isolation level to READ-COMMITTED.
        4.2 The default settings in the MariaDB installations in most distributions use conservative buffer sizes and memory               usage. Cloudera Management Service roles need high write throughput because they might insert many records in the             database. Cloudera recommends that you set the innodb_flush_method property to O_DIRECT.
        4.3 Set the max_connections property according to the size of your cluster:
          4.3.1 Fewer than 50 hosts - You can store more than one database (for example, both the Activity Monitor and Service                 Monitor) on the same host. If you do this, you should:
                a. Put each database on its own storage volume.
                b. Allow 100 maximum connections for each database and then add 50 extra connections. For example, for two                        databases, set the maximum connections to 250. If you store five databases on one host (the databases for                      Cloudera Manager Server, Activity Monitor, Reports Manager, Cloudera Navigator, and Hive metastore), set                      the maximum connections to 550.
          4.3.2 More than 50 hosts - Do not store more than one database on the same host. Use a separate host for each                       database/host pair. The hosts do not need to be reserved exclusively for databases, but each database should                   be on a separate host.
          4.3.3 If the cluster has more than 1000 hosts, set the max_allowed_packet property to 16M. Without this setting, the                 cluster may fail to start due to the following exception: com.mysql.jdbc.PacketTooBigException.
                Although binary logging is not a requirement for Cloudera Manager installations, it provides benefits such as                 MariaDB replication or point-in-time incremental recovery after a database restore. The provided example                       configuration enables the binary log. For more information, see The Binary Log.
                Here is an option file with Cloudera recommended settings:
                [mysqld]
                datadir=/var/lib/mysql
                socket=/var/lib/mysql/mysql.sock
                transaction-isolation = READ-COMMITTED
                # Disabling symbolic-links is recommended to prevent assorted security risks;
                # to do so, uncomment this line:
                symbolic-links = 0
                # Settings user and group are ignored when systemd is used.
                # If you need to run mysqld under a different user or group,
                # customize your systemd unit file for mariadb according to the
                # instructions in http://fedoraproject.org/wiki/Systemd

                key_buffer = 16M
                key_buffer_size = 32M
                max_allowed_packet = 32M
                thread_stack = 256K
                thread_cache_size = 64
                query_cache_limit = 8M
                query_cache_size = 64M
                query_cache_type = 1

                max_connections = 550
                #expire_logs_days = 10
                #max_binlog_size = 100M

                #log_bin should be on a disk with enough free space.
                #Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
                #system and chown the specified folder to the mysql user.
                log_bin=/var/lib/mysql/mysql_binary_log

                #In later versions of MariaDB, if you enable the binary log and do not set
                #a server_id, MariaDB will not start. The server_id must be unique within
                #the replicating group.
                server_id=1

                binlog_format = mixed

                read_buffer_size = 2M
                read_rnd_buffer_size = 16M
                sort_buffer_size = 8M
                join_buffer_size = 8M

                # InnoDB settings
                innodb_file_per_table = 1
                innodb_flush_log_at_trx_commit  = 2
                innodb_log_buffer_size = 64M
                innodb_buffer_pool_size = 4G
                innodb_thread_concurrency = 8
                innodb_flush_method = O_DIRECT
                innodb_log_file_size = 512M

                [mysqld_safe]
                log-error=/var/log/mariadb/mariadb.log
                pid-file=/var/run/mariadb/mariadb.pid

                #
                # include all files from the config directory
                #
                !includedir /etc/my.cnf.d
      5. If AppArmor is running on the host where MariaDB is installed, you might need to configure AppArmor to allow MariaDB          to write to the binary
      6. Ensure the MariaDB server starts at boot:
         6.1 RHEL7 Compatible 
             sudo systemctl enable mariadb
         6.2 RHEL6 Compatible
             sudo chkconfig mariadb on
         6.3 SLES
             sudo chkconfig --add mariadb
         6.4 Ubuntu
             sudo chkconfig mariadb on
      7. Start the MariaDB Server
         7.1 RHEL7
             sudo systemctl start mariadb
         7.2 RHEL6
             sudo service mariadb start
      8. Run /usr/bin/mysql_secure_installation to set the MariaDB root password and other security-related settings. In a new          installation, the root password is blank. Press the Enter key when you're prompted for the root password. For the              rest of the prompts, enter the responses listed below in bold:
         8.1 sudo /usr/bin/mysql_secure_installation
             [...]
             Enter current password for root (enter for none):
             OK, successfully used password, moving on...
             [...]
             Set root password? [Y/n] Y
             New password:
             Re-enter new password:
             [...]
             Remove anonymous users? [Y/n] Y
             [...]
             Disallow root login remotely? [Y/n] N
             [...]
             Remove test database and access to it [Y/n] Y
             [...]
             Reload privilege tables now? [Y/n] Y
             [...]
             All done!  If you've completed all of the above steps, your MariaDB
             installation should now be secure.

             Thanks for using MariaDB!
             
  ## Installing the MySQL JDBC Driver for MariaDB
     RHEL
     1. Download the MySQL JDBC driver from
        wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
     2. Extract the JDBC driver JAR file from the downloaded file. For example:
        tar zxvf mysql-connector-java-5.1.46.tar.gz
     3. Copy the JDBC driver, renamed, to /usr/share/java/. If the target directory does not yet exist, create it. For                 example:
        sudo mkdir -p /usr/share/java/
        cd mysql-connector-java-5.1.46
        sudo cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
        
      SLES
      1. sudo zypper install mysql-connector-java
      
      Ubuntu 
      1. sudo apt-get install libmysql-java

  ## Creating Databases for Cloudera Software
      
     Create databases and service accounts for components that require databases:
     Cloudera Manager Server
     Cloudera Management Service roles:
     Activity Monitor (if using the MapReduce service in a CDH 5 cluster)
     Reports Manager
     Hue
     Each Hive metastore
     Sentry Server
     Cloudera Navigator Audit Server
     Cloudera Navigator Metadata Server
     Oozie
     
     1. Log in as the root user, or another user with privileges to create database and grant privileges:
        mysql -u root -p
        Enter password:
     2. Create databases for each service deployed in the cluster using the following commands
        
        CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
        CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

     3. Create user and Grant Access
        
        MariaDB [(none)]> create user 'amon'@'%' identified by 'amon';
        MariaDB [(none)]> create user 'amon'@'localhost' identified by 'amon';
        MariaDB [(none)]> create user 'amon'@'hostname' identified by 'amon’;
        MariaDB [(none)]> grant all privileges on *.* to 'amon'@'localhost';
        MariaDB [(none)]> grant all privileges on *.* to 'amon'@'%';
        MariaDB [(none)]> grant all privileges on *.* to 'amon'@'hostname';
       
        MariaDB [(none)]> create user 'scm'@'%' identified by 'scm';
        MariaDB [(none)]> create user 'scm'@'localhost' identified by 'scm';
        MariaDB [(none)]> create user 'scm'@'hostname' identified by 'amon’;
        MariaDB [(none)]> grant all privileges on *.* to 'scm'@'localhost';
        MariaDB [(none)]> grant all privileges on *.* to 'scm'@'%';
        MariaDB [(none)]> grant all privileges on *.* to 'scm'@'hostname';
    
        MariaDB [(none)]> create user 'rman'@'%' identified by 'rman';
        MariaDB [(none)]> create user 'rman'@'localhost' identified by 'rman';
        MariaDB [(none)]> create user 'rman'@'hostname' identified by 'rman’;
        MariaDB [(none)]> grant all privileges on *.* to 'rman'@'localhost';
        MariaDB [(none)]> grant all privileges on *.* to 'rman'@'%';
        MariaDB [(none)]> grant all privileges on *.* to 'rman'@'hostname';
 
     4. Confirm that you have created all of the databases
        4.1 SHOW DATABASES;
        4.2 SHOW GRANTS FOR '<user>'@'%';
        
# Installing Cloudera Manager and CDH

## Installing Oracle JDK ( on every Hosts )
   1. RHEL Compatible
      sudo yum install oracle-j2sdk1.8
   2. SLES
      sudo zypper install oracle-j2sdk1.8
   3. Ubuntu
      sudo apt-get install oracle-j2sdk1.8

## Installing Cloudera Manager Server & Agents
    1. Install Cloudera Manager Packages    
      1.1 On Utility Nodes 
          yum -y install cloudera-manager-agent cloudera-manager-daemons cloudera-manager-server
      1.2 On Others Nodes
          yum -y install cloudera-manager-agent cloudera-manager-daemons
      1.3 On every cluster host, configure the Cloudera Manager Agent to point to the Cloudera Manager Server by setting the             following properties in the /etc/cloudera-scm-agent/config.ini configuration file:
          vim /etc/cloudera-scm-agent/config.ini
          server_host : hostname
          server_port : 7182
      1.4 Start the Agents by running the following command on all hosts:
          RHEL 7, SLES 12, Debian 8, Ubuntu 16.04 and higher
            1. sudo systemctl start cloudera-scm-agent
          RHEL 5 or 6, SLES 11, Debian 6 or 7, Ubuntu 12.04 or 14.04
            1. sudo service cloudera-scm-agent start
 
 ## Set up the Cloudera Manager Database
    Cloudera Manager Server includes a script that can create and configure a database for itself. The script can:

    Create the Cloudera Manager Server database configuration file.
    (MariaDB, MySQL, and PostgreSQL) Create and configure a database for Cloudera Manager Server to use.
    (MariaDB, MySQL, and PostgreSQL) Create and configure a user account for Cloudera Manager Server.
    
     1. Syntax for scm_prepare_database.sh
        sudo /opt/cloudera/cm/schema/scm_prepare_database.sh [options] <databaseType> <databaseName> <databaseUser> <password>
     2. Preparing the Cloudera Manager Server Database
        sudo /opt/cloudera/cm/schema/scm_prepare_database.sh <databaseType> <databaseName> <databaseUser>
        sudo /opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h maridbhost --scm-host scmhost scm scm

## Download all jar files that needed by nifi and flink
    
    1. cd /opt/cloudera/csd
    2. wget http://archive.cloudera.com/CFM/csd/1.0.0.0/NIFI-1.9.0.1.0.0.0-90.jar
    3. wget http://archive.cloudera.com/CFM/csd/1.0.0.0/NIFICA-1.9.0.1.0.0.0-90.jar
    4. wget http://archive.cloudera.com/CFM/csd/1.0.0.0/NIFIREGISTRY-0.3.0.1.0.0.0-90.jar
    5. wget --user xxx --ask-password https://archive.cloudera.com/p/csa/1.1.0.0/csd/FLINK-1.9.1-csa1.1.0.0-cdh7.0.3.0-79-1753674.jar
        
## Install CDH and Other Software
   1. Start Cloudera Manager Server
    1.1 RHEL7
        sudo systemctl start cloudera-scm-server
    1.2 RHEL6
        sudo service cloudera-scm-server start
   2. Wait several minutes for the Cloudera Manager Server to start. To observe the startup process, run the following on the       Cloudera Manager Server host:
      sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
      INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.
   3. In a web browser, go to http://<server_host>:7180, where <server_host> is the FQDN or IP address of the host where the         Cloudera Manager Server is running
   4. Log into Cloudera Manager Admin Console. The default credentials are:
      Username : admin
      Password : admin
   5. Accept License
   6. Select Edition
   7. Specify Hosts
   8. Select Repository or modify repository
       http://hostname:8000/cdh6/6.3.2/parcels/
       http://hostname:8000/accumulo-c5/parcels/1.7.2/
       http://hostname:8000/accumulo6/6.1/parcels/
       http://hostname:8000/kafka/parcels/latest/
       http://hostname:8000/kudu/parcels/latest/
       http://hostname:8000/spark2/parcels/2.3.0.cloudera3/
       http://hostname:8000/p/csa/1.1.0.0/parcels/
       http://hostname:8000/CFM/parcels/1.0.1.0/
       http://hostname:8000/sqoop-connectors/parcels/latest/
       http://hostname:8000/sqoop-netezza-connector1/latest/
   9. Accept JDK License
   10. Enter Login Credentials
   11. Install Agents
   12. Install Parcels
   13. Inspect Cluster

## Install Nifi and Flink
    
    1. login to cloudera manager
    2. add service
    3. add nifi and flink repository
        http://hostname:8000/p/csa/1.1.0.0/parcels/
        http://hostname:8000/CFM/parcels/1.0.1.0/
    4. download nifi and flink
    5. distributes nifi and flink
    6. activates nifi and flink
    7. Setting up your HDFS Home directory
        > HADOOP_USER_NAME=hdfs hdfs dfs -mkdir /user/$USER_NAME
        > HADOOP_USER_NAME=hdfs hdfs dfs -chown $USER_NAME:$USER_NAME /user/$USER_NAME
    8. Setting the Java executable for the Flink client
      8.1 setup java home for root user
          touch vim ~/.bash_profile
          JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
          PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
          save and exit
          source ~/.bash_profile
      8.1 setup java home for flink user
          su flink
          touch vim ~/.bash_profile
          JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
          PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
          save and exit
          source ~/.bash_profile
          
          


      
    
    


 
    
    
    
