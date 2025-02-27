# mySQL

## What is it and Installation Process

> It's an open source database management system. It implements the relational model and uses Structured Query Language (aka SQL) to manage its data.

# Install & pre-requisites

- Ubuntu server with non-root administrative user and firewall configured with **UFW**.

1. Install `mysql-server` package.

```bash
sudo apt install mysql-server
```

2. Start MySQL.

```bash
sudo systemctl start mysql.service
```

Next is the configuration. We'll use the DBMS's security script as it enables us to change some of the less secure options for things like remote logins and sample users.

1. Open up MySQL prompt.

```bash
sudo mysql
```

then:

```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

The command above changes the root user's auth method to one that uses a password.

Now exit:

```bash
mysql> exit
```

Now run `mysql_secure_installation`.
