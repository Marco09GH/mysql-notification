# mysql-notification

A simple example of using a user defined function (UDF) in mysql to make real-time notifications on a table change. This project concists of a mysql plugin that setups a server socket that receives messages from a trigger connected to INSERT, UPDATE, DELETE operations on a specific table in the database. The server will then send a message to a nodejs server that in turn will bounce this notification to any connected http client over a websocket.

# Compiling

- You first need to build your shared library using:

```
$ gcc -c -Wall -fpic mysql-notification.c -I/path/mysql/headers
$ gcc -shared -o mysql-notification.so mysql-notification.o
```

Notice that you'll need to have the mysql headers installed on your system. 
Using linux this can be done by running:

> $ sudo apt-get update
> $ sudo apt-get install libmysqld-dev

On OSX using brew you can install the mysql package which contains the needed headers.

# Installation

- Install modules from package.json

> $ npm install

- Setup your user defined function (UDF) by adding the shared library into mysqls plugin folder, this folder 
can be located by executing `SHOW VARIABLES LIKE 'plugin_dir';` from the mysql client.

> $ cp mysql-notification.so /usr/local/mysql/lib/plugin/.

- Tell mysql about the UDF

> $ CREATE FUNCTION MySQLNotification RETURNS INTEGER SONAME 'mysql-notification.so';

- Create triggers for INSERT/UPDATE/DELETE

```
$ DELIMITER @@
$ CREATE TRIGGER <triggerName> AFTER INSERT ON <table> 
  FOR EACH ROW 
  BEGIN 
    SELECT MySQLNotification(NEW.id, 2) INTO @x; 
  END@@
$ CREATE TRIGGER <triggerName> AFTER UPDATE ON <table>
  FOR EACH ROW 
  BEGIN 
    SELECT MySQLNotification(NEW.id, 3) INTO @x; 
  END@@
$ CREATE TRIGGER <triggerName> AFTER DELETE ON <table>
  FOR EACH ROW 
  BEGIN 
    SELECT MySQLNotification(OLD.id, 4) INTO @x; 
  END@@
$ DELIMITER ;
```

# Running

> $ node server.json

- Go to address http://127.0.0.1:999 in your browser and start receiving notifications from your database.

