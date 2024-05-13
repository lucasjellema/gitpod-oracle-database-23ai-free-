# gitpod-oracle-database
Run Gitpod workspace with Oracle Database Free 23ai (23.4.0.0.0)


[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/lucasjellema/gitpod-oracle-database-23ai-free-)


After approximately 5-8 minutes Oracle Database Free 23ai is running in a container called *oracle-database-free*. (the download followed by starting up the database takes a while)

You can access the database at port 1521 on localhost - using user SYS (as SYSDBA) and password TheSuperSecret1509! or user developer with password devPassword!

You can connect to the Oracle Database server by running a SQL*Plus command from within the container - using

```
docker exec -it oracle-database-free sqlplus sys/"TheSuperSecret1509!" as sysdba
```

or connect as the admin of Pluggable Database FREEPDB1:

```
docker exec -it oracle-database-free sqlplus pdbadmin/"TheSuperSecret1509!"@FREEPDB1
```


The database is started using the Container image made available here: [Oracle Database Free Container Image Details](https://container-registry.oracle.com/ords/f?p=113:4:2674354842458:::4:P4_REPOSITORY,AI_REPOSITORY,AI_REPOSITORY_NAME,P4_REPOSITORY_NAME,P4_EULA_ID,P4_BUSINESS_AREA_ID:1863,1863,Oracle%20Database%20Free,Oracle%20Database%20Free,1,0&cs=33IH4gtTe8pk7MyXUDJbv-zEuD6C-pIcPJZthvsM3AQkjKMNpmy5OOSHIXXVkud3J4eER6KDWOqq-ev63NHJn_A).



You can use the VS Code Oracle Developer Tools extension for VS Code - which is preconfigured with a database connection for the SYS account.

SQLcl command line tool (see [SQLcl introduction](https://www.oracle.com/database/sqldeveloper/technologies/sqlcl/) for details) has also been installed and can be accessed using:

```
sql sys/"TheSuperSecret1509!"@localhost:1521/FREE as sysdba 
```  

A database user has been setup with username DEV and password DEV_PW and another one with APP/APPPW - both in PDB `FREEPDB1`. A connection for this user is created in VS Code Oracle Developer Tools. Connect through SQLcl can be done using:

```
sql DEV/DEV_PW@localhost:1521/FREEPDB1 
```  


## Reusing the Existing Database
If the database is started using a host system directory mounted at an /opt/oracle/oradata location inside the container, as explained in the Custom Configuration section under "Mounting docker volume/host directory for database persistence", then the data files remain persistent even after the container is destroyed. Another container with the same data files can be started by reusing the host system directory. Startup will then be quite fast!

To reuse this directory on the host system for the data volume, run the following commands:

```
docker run -d --name oracle-database-free -v <writable_host_directory_path>:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
```

f a host system directory is mounted on the /opt/oracle/oradata location, two things can happen :

If the datafiles are present in the host system directory which is being mounted, then these files will overwrite the files present at /opt/oracle/oradata (in the container) and the database startup will be very fast.

If host system directory doesn't contain the datafiles then a new database setup will begin. It takes a significant amount of time (approximately 10 minutes) to set up a fresh database. 

Note:* Cleaning up a mounted host directory can be done by changing into the directory location and executing rm -rf * .*


## Running Scripts After Setup and On Startup
The Container images can be configured to run scripts after setup and on startup. Currently, .sh and .sql extensions are supported. For post-setup scripts, mount the volume /opt/oracle/scripts/setup to include scripts in this directory. For post-startup scripts, mount the volume /opt/oracle/scripts/startup to include scripts in this directory.

After the database is set up or started, the scripts in those folders are run against the database in the container. SQL scripts are run as SYSDBA, and shell scripts are run as the current user. To ensure proper order for running scripts, Oracle recommends that you prefix your scripts with a number. For example: 01_users.sql, 02_permissions.sql, and so on.

Note: The startup scripts are run after the first time that the database setup is complete.

The following example mounts the local directory /home/oracle/myScripts to /opt/oracle/scripts/startup, which is then searched for custom startup scripts:

  $ docker run -d --name <oracle-db> -v
  /home/oracle/myScripts:/opt/oracle/scripts/startup
  container-registry.oracle.com/database/free:latest