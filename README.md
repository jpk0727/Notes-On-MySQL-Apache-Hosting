# Notes-On-MySQL-Apache-Hosting

This repository is to hold notes for the Swarthmore Department of Engineering on how to use the CentOS server to host a database and serve a web application suing Apache. This project was created after Jess Karol and Connie Bowen figured out these tools for their E90. 

## MySQL ##
- Start mysql from command line. Either an existing user or the root account can
execute this command. -u flag is followed by username (in this case root) and -p flag will prompt for a password
    mysql -u root -p
- Then create a username with permisions for the person that wishes to use the database. 
    mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
        ->     WITH GRANT OPTION;

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


    
    
