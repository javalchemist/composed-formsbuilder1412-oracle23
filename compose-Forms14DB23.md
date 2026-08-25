<div align="center"><h4 style="margin-bottom:3px">Forms Builder 14.1.2 platform</h4><h4>-based on docker compose-</h4></div>

### 1. intro

**Purpose:**

- building Forms Builder 14.1.2 image based on a `dockerfile` . Image should support easy starting of a development platform with Oracle database.

-  create a *docker compose* file  for a  development platform consisting of : Forms Builder 14.1.2. container and Oracle 26 database networked together, ready for connection.

- easy start and stop of platform , with little to no configuration

This project is intended to **offer a working platform for testing and developing for Debian distribution of Linux** operating system. To work in Windows environment, there should be some adjustments , as for example a slightly different configuration for Forms Builder container. 

  

### 2. quick start

You will need : Oracle account allowing download of binary files from `edelivery.oracle.com`, locally storage space and installed docker.

- test docker is working

- use script and `Dockerfile.ol9` at [Oracle on github](https://github.com/oracle/docker-images/tree/main/OracleJava/21)  and **build image of oracle linux 9 with jdk21** using command line :

  ````shell 
  $ ./build.sh 9
  ````

  Result will be an image called `oracle/jdk:21-ol9`

- on the host **issue permission to X window manager** :

  ````shell
  $ xhost + local:
  ````

- on the host machine create a folder that will host Foms Builder installation. Here you will clone this project.

- **connect to `edelivery.oracle.com`** using your Oracle account  . Installation package for Forms Builder can be found when search for "*Oracle Fusion Middleware 14c (14.1.2.0.0) Forms and Reports for Linux x86-64 for (Linux x86-64)*". 

- unzipp downloaded archive to get binary installation package **`fmw_14.1.2.0.0_fr_linux64.bin`. Place this file in folder `installer`** together with `responseFile`.

- from `PROJECT_HOME` folder, where you have `compose.yaml` file  **start the platform** with

  ````shell
  $ docker compose up -d
  ````

- after Forms Builder window shows on the host , **connect to database** using 

  - User Name = `appUser`
  - Password =  `userPass`
  - Database = `fb14db`

- **place FMB files  you want** to have available , in host folder `<PROJECT_HOME>/external_files`



### 3. database container

This is based on the very handy and preciselly crafted image of Oracle database  available on  [github](https://github.com/gvenzl/oci-oracle-free/blob/main/README.md) made by [Gerald Venzl](https://github.com/gvenzl) .  This projects will use the `slim` version  of Oracle database version 23.26



### 4. base image = oracle linux 9 + JDK21

Two factors are to be taken into considerance:

- Oracle already provide   on [github](https://github.com/oracle/docker-images/tree/main/OracleJava/21) the `build.sh` and coresponding `dockerfile` for  an image with oracle linux 9 and JDK21
- nor `docker compose` neither commands accepted by a `dockerfile` are able to naturally control launching of a script file that builds this image.

The logical **conclusion** is this : in this case is best to obtain base image ( "oracle linux 9 + jdk21")  by first calling a single time  the [`build.sh`](https://github.com/oracle/docker-images/blob/main/OracleJava/21/build.sh). This will remain *the single isolate process step*.



### 5. dockerfile for Forms Builder 14.1.2

It will create an image for a container with all settings necessary to enable connecting of Forms Builder with database .

**Installation require an Oracle binary installation package** that can be obtain with an Oracle account, from `edelivery.oracle.com`  where can be found if search for "*Oracle Fusion Middleware 14c (14.1.2.0.0) Forms and Reports for Linux x86-64 for (Linux x86-64), 1.3 GB*". 

Unzipped file should be called  `fmw_14.1.2.0.0_fr_linux64.bin`  and will be placed locally in `<PROJECT_HOME>/installer`  folder.

The **connect string  is "fb14db"**  and is inserted in both `$ORACLE_HOME/formsInst1/tnsnames.ora`  and in `$ORACLE_HOME/network/admin/tnsnames.ora` to be used with SQL-client installed if necessary .

````properties
fb14db =
(DESCRIPTION =
 (ADDRESS =
   (PROTOCOL = TCP)
   (HOST = db-fb1412-platform)
   (PORT = 1521))
 (CONNECT_DATA =
   (SERVER = DEDICATED)
   (SERVICE_NAME = freepdb1)))
````

Database is already configure with an **account that can be used by Forms Builder to connect** :

- User Name = `appUser`
- Password =  `userPass`
- Database = `fb14db`

**Important notice**:  installation file `fmw_14.1.2.0.0_fr_linux64.bin` should have "644" permission, allowing everybody to read this file.



### 6. composing file for the platform

Command to build artefacts and start containers implies following directory structure :

````tex
proiect ----- formsBuilder1412
		  |		|--- Dockerfile
		  |
		  |--- installer
		  |		|--- fmw_14.1.2.0.0_fr_linux64.bin
		  |		|--- responseFile
		  |----ol9-jdk21
		  			|--- Dockerfile.ol9
		  			|--- build.sh
		  |--- external_files
		  			|--- FMB files
````

There is a `bind` **volume specifically defined to enable sharing files between host and Forms Builder container**, for exampe Forms FMB files. This **directory called `external_files`** is a subdirectory of `<PROJECT_HOME>` . Is best to exist before starting `docker compose` command or docker will create it with owner `root`.

Keep in mind that this platform is intended for development and testing only. This make keeping secrets an effort with not much return value.This is why the connection data for database or the database system password are all accesible.



### 7. to do

- `entrypoint.sh` for Forms Builder container, that probes database connection readiness
- editing buffer sharing with host
- reduce image space
