# MySQL(3306)

## About

**MySQL** is an open-source SQL relational database management system developed and supported by Oracle. It works according to the _**client-server**_ principle and consists of a MySQL server and one or more MySQL clients. The database is controlled using the [SQL database language](https://www.w3schools.com/sql/sql\_intro.asp). The data is stored in tables with different columns, rows, and data types. These databases are often stored in a single file with the file extension `.sql`, for example, like `wordpress.sql`



## Default Configuration

{% hint style="info" %}
Managing SQL databases and their configurations is an extensive field. It's so comprehensive that specialized professions, such as database administrators, focus almost exclusively on databases. These systems can grow rapidly and their planning can become quite complex.
{% endhint %}

```shell-session
$ sudo apt install mysql-server -y
$ cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

### Dangerous Settings

Like any server configuration, anything can go wrong if server administrator oversee a vulnerable setting. The settings below are the "main" options that are security-relevant:

| <mark style="color:red;">**user**</mark>               | Sets which user the MySQL service will run as.                                                               |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| <mark style="color:red;">**password**</mark>           | Sets the password for the MySQL user.                                                                        |
| <mark style="color:red;">**admin\_address**</mark>     | The IP address on which to listen for TCP/IP connections on the administrative network interface.            |
| <mark style="color:red;">**debug**</mark>              | This variable indicates the current debugging settings                                                       |
| <mark style="color:red;">**sql\_warnings**</mark>      | This variable controls whether single-row INSERT statements produce an information string if warnings occur. |
| <mark style="color:red;">**secure\_file\_priv**</mark> | This variable is used to limit the effect of data import and export operations.                              |

## Footprinting

### Nmap

```shell-session
$ sudo nmap <TARGE IP> -sV -sC -p3306 --script mysql*
```

{% hint style="danger" %}
we should always be careful with the results we get from any automated tool. In this case, there's a big chance we run into false positives when Nmap scans the port and returns usernames marked as valid when they're actually not.\
Remember that the server configuration is what determines the returning values these automated tools get. So a simple return status could force the tool to mishandle that information and get us "wrong" output.
{% endhint %}

### Interacting With The Server

```shell-session
$ mysql -u <USER> -p<PASSWORD> -h 10.129.14.128
```

|                      **Command**                     | **Description**                                                                                                                                                                                                                                                       |
| :--------------------------------------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|    `mysql -u <user> -p<password> -h <IP address>`    | Connect to the MySQL server. <mark style="color:red;">There should</mark> <mark style="color:red;"></mark><mark style="color:red;">**not**</mark> <mark style="color:red;"></mark><mark style="color:red;">be a space between the '-p' flag, and the password.</mark> |
|                   `show databases;`                  | Show all databases.                                                                                                                                                                                                                                                   |
|                   `use <database>;`                  | Select one of the existing databases.                                                                                                                                                                                                                                 |
|                    `show tables;`                    | Show all available tables in the selected database.                                                                                                                                                                                                                   |
|             `show columns from <table>;`             | Show all columns in the selected database.                                                                                                                                                                                                                            |
|               `select * from <table>;`               | Show everything in the desired table.                                                                                                                                                                                                                                 |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.                                                                                                                                                                                                                      |

