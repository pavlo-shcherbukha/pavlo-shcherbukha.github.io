---
layout: post
title:  "Установка MySql на VM з ОS Ubunta"
date:   2022-04-30 12:04:51
categories: [ubunta, mysql]
permalink: posts/2022-04-30/ubunta-setup-mysql/
published: true
---

# Установка MySql  на  vm UBUNTA

Для тестування і розуміння vm  та всіх приколів побудуємо багато рівневе application
Центром всього цьго буде БД mysql 

## Установка mysql сервер на UBUNTU

Взято по матеріалах з [Phoenixnap home contact support blog How to Install MySQL on Ubuntu 20.04](https://phoenixnap.com/kb/install-mysql-ubuntu-20-04).

1. Open the terminal and run the following command:

```bash
sudo apt update
```
2. Enter your password and wait for the update to finish.

3. Next, run:

```bash
sudo apt upgrade
```

4. Enter Y when prompted to continue with the upgrade and hit ENTER. Wait for the upgrade to finish.
Step 2: Install MySQL

1. After successfully updating the package repository, install MySQL Server by running the following command:

```bash
sudo apt install mysql-server
```
2. When asked if you want to continue with the installation, answer Y and hit ENTER.
Install MySQL server on Ubuntu.

The system downloads MySQL packages and installs them on your machine.

Note: If you only want to connect to a remote MySQL server instead of hosting a database on your machine, install only the MySQL Client by running:

```bash
sudo apt install mysql-client

```

3. Check if MySQL was successfully installed by running:

```bash
mysql --version

````

Check MySQL version on Ubuntu.

The output shows which version of MySQL is installed on the machine.

Note: If you are using Windows, learn how to install and configure MySQL on a Windows Server.
Step 3: Securing MySQL

The MySQL instance on your machine is insecure immediately after installation.

1. Secure your MySQL user account with password authentication by running the included security script:

```bash
sudo mysql_secure_installation
```
Secure MySQL server by setting up an authentication password.

2. Enter your password and answer Y when asked if you want to continue setting up the VALIDATE PASSWORD component. The component checks to see if the new password is strong enough.

3. Choose one of the three levels of password validation:

    0 - Low. A password containing at least 8 characters.
    1 - Medium. A password containing at least 8 characters, including numeric, mixed case characters, and special characters.
    2 - Strong. A password containing at least 8 characters, including numeric, mixed case characters, and special characters, and compares the password to a dictionary file.

Enter 0, 1, or 2 depending on the password strength you want to set. The script then instructs you to enter your password and re-enter it afterward to confirm.

Any subsequent MySQL user passwords need to match your selected password strength.

Note: Even though you are setting a password for the root user, this user does not require password authentication when logging in.

The program estimates the strength of your password and requires confirmation to continue.

4. Press Y if you are happy with the password or any other key if you want a different one.
MySQL password validation script estimates password strength.

5. The script then prompts for the following security features:

    Remove anonymous users?
    Disallow root login remotely?
    Remove test database and access to it?
    Reload privilege tables now?

The recommended answer to all these questions is Y. However, if you want a different setting for any reason, enter any other key.

For example, if you need the remote login option for your server, enter any key other than Y for that prompt.

Note: Check out our tutorial if you want to create a new MySQL user and grant privileges.
Step 4: Check if MySQL Service Is Running

Upon successfully installing MySQL, the MySQL service starts automatically.

Verify that the MySQL server is running by running:

```bash
sudo systemctl status mysql
```
The output should show that the service is operational and running:
How to check if MySQL service is running in Ubuntu.
Step 5: Log in to MySQL Server

Finally, to log in to the MySQL interface, run the following command:

```bash
sudo mysql -u root
```
Login to MySQL Shell in the Ubuntu terminal.

Now you can execute queries, create databases, and test out your new MySQL setup. Take a look at o



6. Зробити можливем підклчення до mysql з віддаленої машини

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

та замінити 

```text

bind-address            = 127.0.0.1
mysqlx-bind-address     = 127.0.0.1

```

 на 


```text

bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```

