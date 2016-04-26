# Notes-On-MySQL-Apache-Hosting

This repository is to hold notes for the Swarthmore Department of Engineering on how to use the CentOS server to host a database and serve a web application using Apache. This project was created by Jess Karol after he figured out how to use these tools for their E90. Feel free to contribute more instructions to this page to 
record information so future engineers have an easier time. 

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

    mysql> CREATE DATABASE dbname;

Then you can grant privilages to that database like this:

    mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
    mysql> GRANT ALL PRIVILEGES ON dbname.* TO 'monty'@'%'
        ->     WITH GRANT OPTION;

- Instead of using the MySQL Command Line interface, it may be easier to download
the MySQL workbench IDE from the internet and use this tool to run the same commands. This tool has many other features that may be helpful in database management. For instance, it is very easy to test SQL statements. Here is the link to download the application: [mysql workbench](https://dev.mysql.com/downloads/workbench/) 
    - to create a connection to the database from the MySQL workbench, you will 
    enter the host: "fubini.swarthmore.edu" and you can use port "3306", which 
    is the default. 

### MySQL Python ###
- Many programing languages have libraries to estabilish a connection with MySQL
and execute SQL statements. I will breifly explane some usefull parts of the 
python library. You will need to isntall the python package manager (pip) on 
the machine you are using if it is not already. This may be helpful if you are
using a raspberry pi with an internet connection to sumbit data from the GPIO pins
to a SQL database. 

- Install MySQL-python

'''
sudo pip install MySQL-python
'''

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

## DJANGO WEB APPLICTION ##
Django is a framework for creating web applications that uses mostly the python language. This framework
has good documentation, easily interfaces with any database, and is fairly streight forward. 
The easiest way to start a django project is by using one of the many django start templates.
I recommend choosing a simple django skeleton because the more complicated starter templates
are designed for much larger scale applications than what most Swarthmore projects will be using.
Here is a link to the starter project that I used. [django-starter-template](https://github.com/fasouto/django-starter-template)

The only complication with using django template is that it requires python 2.7, which most operating systems
use. However, the centOS server does not natively run python 2.7, so I will show you how to work
around this later in the Apache section. 

To tie your Django application to a database, add the following lines to the development.py 
configuration file

```python
TABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'dbname',                      # Or path to database file if using sqlite3.
        'USER': 'username',                      # Not used with sqlite3.
        'PASSWORD': 'password',                  # Not used with sqlite3.
        'HOST': '130.58.84.237',                      # Set to empty string for localhost. Not used with sqlite3.
        'PORT': '3306',                      # Set to empty string for default. Not used with sqlite3.
    }
```

- run the application to see if it is working by entering the following command into the terminal. Make sure
you are in the project root directory that contains the manage.py file.

    python manage.py runserver

- You can now go to a browser and visit localhost:8000 to view the website. You are ready to develop the views
and functionality you want. One important feature of django is the models, which is described below.
### DJANGO MODELS ###

Once the database is set up properly by simply entering the connection information into the development.py 
settings file, you can start using the models to build the database. 

Models are used as an interface between SQL and your web application. You should create models for db tables where
the data is used on the website. Here is an example of a simple model, which would be placed in the /apps/base/models.py file

```python
from django.db import models

class students(models.Model):
    name = models.CharField()
    class_yr = models.IntegerField()
```

This creates a model for a table that will hold information about a student, including the name and class year.
Now return to the base directory for the django project and run the following commands to create the tables
in the database. 

```
>> python manage.py makemigrations
>> python manage.py migrate
```

This will take the model that you created, turn it into SQL and exectue the command over the database connection.
You can interface with these models in the views.py to send data to html pages, and have website users send data
back to the database using a ModelForm. In other words, the models are a layer between the database and the web 
application

## APACHE and Serving the Web App ##

Fortunately, using the Django starter template already prepares the web application to be served using mod_wsgi.
The first steps in serving the website are creating a virtual environment on the CentOS server that will
contain all the dependencies for the web application. To create a new environment run the following command
on the server.

    virtualenv projectname

cd into the newly created folder and then activate the shell for the environment. This will allow you to install
dependencies into the environment without them also installed on the rest of the machine. Interesting but importantly,
the fubini server is already a virtual server, so we are creating a virtual environment on a virtual server. Before we
activate the virtual shell, however, we must make sure we are first using the python 2.7 which is required for 
the Django application. 

    scl enable python27 bash
    python -V

If the last command shows python 2.7 then you are now using the correct python version.

    source /bin/activate

Now clone you Django webapp repo from github and install all the required dependenices. (This is assuming
you developed the web app on another computer and saved the code to a github account). I had trouble
using the swarthmore github, so I used a private account. This may include installing the python 2.7, which 
will be installed in the lib folder in the environment.

To set up my httpd django configuration, I added the fallowing to the django.conf file in the 
httpd config folder

    ```
    alias /static /home/jess/growEnv/growApp/grow/static
    <Directory /home/jess/growEnv/growApp/grow/static>
        order allow,deny
        allow from all
    </Directory>

    <Directory /home/jess/growEnv/growApp/grow>
        <Files wsgi.py>
            order allow,deny
            allow from all
        </Files>
    </Directory>

    WSGIDaemonProcess grow python-path=/home/jess/growEnv/growApp:/home/jess/growEnv/lib/python2.7/site-packages
    WSGIProcessGroup grow
    WSGIScriptAlias /grow /home/jess/growEnv/growApp/grow/wsgi.py
    WSGISocketPrefix /var/run/wsgi
    ```

You may need to modify or add to this for a different Django project. Notice, we are not using the native python library,
but the python library that is installed within the virtual env. 

The following link is a tutorial on how to host a djanog application using apache, but it is not specific to CentOS, and 
does not assume a project has already been created [link to tutorial](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-ubuntu-14-04)



                                











    
    
