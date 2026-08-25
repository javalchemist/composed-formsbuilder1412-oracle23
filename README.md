###  Forms Builder 14.1.2.0 docker composed with Oracle database 23 to work with GUI on linux host

Intention is to make a handy platform meant for testing and developing Oracle forms compatible with Forms 14.1.2.  There are several guide lines that shape the implementation:

- to be simple to install and even simpler to run
- configuration is straightforward and opinionated
- to access specific Oracle Forms files  (as FMB, PLL) you have a directory in `<PROJECT_HOME>`
- solution is tested on linux Mint but compatibility are generous .



### requirements

- about 15-20GB host storage space
- Oracle account allowing download of binary files from [edelivery.oracle.com](https://edelivery.oracle.com)
- docker installed
- linux host



### the process:

- download files to build a base image from [Oracle on github](https://github.com/oracle/docker-images/tree/main/OracleJava/21) 
- download binary installation package for Forms Builder from Oracle
- launch 1 unix script file for the base image
- one command line in terminal on host to give permissions for GUI 
- `docker compose up -d` gives you the platform and start Forms Builder GUI
- connect from Forms Builder to Oracle database



### result

You will be able to:

- start database and Forms Builder to *open, edit and save an Oracle Forms files and libraries*
- connect to database with username, password and connection string
- *Forms Builder will show* on screen using windows on *host window system manager*
- open Forms FMB files from your specific folder on host.

