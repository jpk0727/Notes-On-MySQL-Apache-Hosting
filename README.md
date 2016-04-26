# Notes-On-MySQL-Apache-Hosting

This repository is to hold notes for the Swarthmore Department of Engineering on how to use the CentOS server to host a database and serve a web application suing Apache. This project was created after Jess Karol and Connie Bowen figured out these tools for their E90. 

## MySQL ##
- Start mysql from command line. Either an existing user or the root account can
execute this command. -u flag is followed by username (in this case root) and -p flag will prompt for a password

```
    mysql -u root -p
```

- Then, once a MySQL command line is reached, create a username with permisions for the person that wishes to use the database. 

```
    mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
        ->     WITH GRANT OPTION;
```

This statement will create a user 'monty' with password 'some_pass' that has 
all privilages on all schemas. This is not exactly perferable because 
likely we will not want to grant all privilages on all the schemas. A schema
is a directory of tables within a database. Instead you could create a schema 
for the user and then grant privilages only within that schema like this:

    CREATE DATABASE dbname;

Then you can grant privilages to that database like this:

    mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
    mysql> GRANT ALL PRIVILEGES ON dbname.* TO 'monty'@'%'
        ->     WITH GRANT OPTION;

- Instead of using the MySQL Command Line interface, it may be easier to download
the MySQL workbench IDE from the internet and use this tool to run the same commands. This tool has many other features that may be helpful in database management. For instance, it is very easy to test SQL statements. 
    - to create a connection to the database from the MySQL workbench, you will 
    enter the host: "fubini.swarthmore.edu" and you can use port "3306", which 
    is the default. 

### MySQL Python ###
- Many programing languages have libraries to estabilish a connection with MySQL
and execute SQL statements. I will breifly explane some usefull parts of the 
python library. You will need to isntall the python package manager (pip) on 
the machine you are using if it is not already.
    - Install MySQL-python

    sudo pip install MySQL-python

    - Establishing a connection: place the following code at the top of a 
    python script.
```python
    db = MySQLdb.connect(host="fubini.swarthmore.edu",
            port = 3306,
            user="jess",
            passwd="aeroponic_growth",
        db="grow")
    cur = db.cursor()
```
    - Execute a simple select statement.
```python
    sql = "select * from dbname.table"
    num_rows = cur.execute(sql)
    if (num_rows > 0):
        data= cur.fetchall()
```
    - Execute a simple insert statement
```python
    sql = "insert into dbname.table (var1, var2) values (%(var1)s,%(var2))"
    values = {
        'var1': var1
        'var2': var2
    }
    cur.execute(sql, values)
    db.commit()
```
    - close database connection when done
```python
    db.close()
```








    
    
