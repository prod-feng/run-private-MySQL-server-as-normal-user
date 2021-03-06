# run-private-MySQL-server-as-normal-user

The following is my personal experience, use at one's own risk.

To run a private instance of ```mysqld``` MySQL server as a normal user sometimes may needed. Here is what I did on Centos 7.

There are several ways to do so. One way is to use the MySQL package insatlled by your sysadmin, while create the database settings and files in your own HOME direcory; the other way is to install your own copy of MySQL in your HOME direcory and configure it. You can downlwd the rpm files, and install them into your folders; or you can download the source files and compile/install it.

**(1) Download and install MySQL server and client.**

What I did is to download the rpm files, from ```http://mirror.centos.org/centos/7/os/x86_64/Packages/``` . Here one should make sure the pre-compiled version is compatible with the local OS. The 3 files are:

```mariadb-5.5.65-1.el7.x86_64.rpm ```

```mariadb-libs-5.5.65-1.el7.x86_64.rpm```

```mariadb-server-5.5.65-1.el7.x86_64.rpm ```

After downlowd, expand the files and save the pre-compiled files inside them into your HOME directory:


```[feng@server1 ~]$ mkdir /home/feng/mysql```

```[feng@server1 ~]$ cd /home/feng/mysql```

```[feng@server1 ~]$ rpm2cpio   ~/Download/mariadb-server-5.5.65-1.el7.x86_64.rpm | cpio -idmv```

Do the same to the other 2 rpm files. After this is done, you will see the pre-compiled files in the automatically generated sub-folders:


```[feng@server1 ~]$ ls  /home/feng/mysql```

```drwxrwxr-x 5 feng feng 95 Oct 12 11:24 etc```

```drwxrwxr-x 7 feng feng 93 Oct 12 11:24 usr```

```drwxrwxr-x 5 feng feng 55 Oct 12 11:24 var```

Now, I have my own pre-compiled MySQL server and client ready to be run. Before to start to run, I need to do some configurations.

First, set the runtime environment to let the OS to know where my MySQL files are.  I add 3 lines below into my ```.bashrc``` file. 

```export LD_LIBRARY_PATH=/home/feng/mysql/usr/lib64:$LD_LIBRARY_PATH```

```export PATH=/home/feng/mysql/usr/libexec:/home/feng/mysql/usr/bin:/home/feng/mysql/usr/sbin:$PATH```

```export PHP_INI_SCAN_DIR=/home/feng/mysql/etc/php.d```

Source the updated ```.bashrc``` file to update, and test if it is alright:

```[feng@server1 ~]$ which mysqld_safe```

```~/mysql/usr/bin/mysqld_safe```

It is good, the OS now knows where my private MySQL is.

**(2) Configure MySQL server and client.**

Now to configure the MySQL server. There are many ways to do so, I will focus on the easiest one as below: 

Edit my private ```/home/feng/.my.cnf``` file, add following lines:


```[client]```

```port=3306```

```socket=/home/feng/mysql/var/lib/mysql/mysql.sock```

``` ```

```[mysqld]```

```port=3306```

```datadir=/home/feng/mysql/var/lib/mysql```

```socket=/home/feng/mysql/var/lib/mysql/mysql.sock```

```symbolic-links=0```

```user=feng```

```basedir=/home/feng/mysql/usr```

```plugin-dir=/home/feng/mysql/usr/lib64/mysql/plugin```

``` ```

```[mysqld_safe]```

```log-error=/home/feng/mysql/var/log/mariadb/mariadb-feng.log```

```pid-file=/home/feng/mysql/var/run/mariadb/mariadb.pid```


```!includedir /home/feng/mysql/etc/my.cnf.d```


I can also edit my private ```/home/feng/mysql/etc/my.cnf``` file, add/update the above lines into it. While everytime when I need to run MySQL, I have to explicitly direct MySQL to use the configuration file by ``` mysqld_safe --defaults-file=/home/feng/mysql/etc/my.cnf ```. And also when I try to run MySQL client,  I have to run "mysql -u feng -p --socket=/home/feng/mysql/var/lib/mysql/mysql.sock". MySQL seems by default search ```/etc/my.cnf``` and ```.my.cnf```, and will search the  ```--defaults-file``` if it is explicitly set.

Beside, the ```/home/feng/mysql/var/run/mariadb/``` needs to be made, since it is not generated by default. Using command ```mkdir -p /home/feng/mysql/var/run/mariadb/``` should work.

Before to start to run the MySQL server, one might need to run the following command to initialize the MySQL system databases;

```mysql_install_db --no-defaults --basedir=/home/feng/usr --datadir=/home/feng/var/lib/mysql```

**(3) Now I can run my own MySQL server simply by issuing command:**

```mysqld_safe &```

Finnaly, if the OS has SELinux and Firewall enabled, the port for MySQL may need to be set and enabled there.
