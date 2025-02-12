# The Milk Problem

This is how I setup my Windows 11 environment for The Milk Problem coding exercise. 

## General Notes

- There are many different ways you can setup your environment, this is just how I chose to do it so I thought it would be helpful to share.
- You should use Command Prompt when running the commands listed in this guide. Some of the commands will not work if using PowerShell.
- I chose to use DBeaver as my DBMS as opposed to pgAdmin. During the coding exercise, I really only used it as a convenient way to view the data in the databases. If you prefer to use pgAdmin or psql commands in your terminal, feel free to do that instead. Everything else in this guide should still work regardless of which DBMS you choose.

## Prerequisites

This guide assumes that you already have the following software installed.

- IntelliJ IDEA Community Edition (2024.3.2.2)
- Java Development Kit (17.0.2)
- Gradle (8.12.1)

You do not need to use the exact versions listed above. These are just the versions that I used. I will note that I ran into some conflicts with Gradle when using some of the newer versions of Java, so if you encounter similar issues I would suggest trying an earlier JDK version.

## Additional Software

You will need to install some additional software for this exercise.

- PostgreSQL
- Flyway
- DBeaver

### Install PostgreSQL

1. Download and install PostgreSQL. 
    - On the Select Components step, I chose not to install pgAdmin or Stack Builder. For all other options I just used the defaults.
    - Be sure to make note of the password that you enter for the postgres superuser.
    - After installing, the postgresql service should start running automatically. You can validate this by opening Task Manager, going to the Services tab, and searching for postgres.

2. Add the Postgres bin directory to your PATH environment variable. In my case, the path was `C:\Program Files\PostgreSQL\17\bin`.

3. Validate your installation by running `psql --version` in Command Prompt. 

### Install Flyway

1. Download the Flyway CLI. 
    - I downloaded my copy from the [Redgate Installers](https://documentation.red-gate.com/fd/installers-172490864.html) page. 
    - Note that there are 3 different products: Flyway Desktop, Flyway Docker, and Flyway CLI. I chose to just download the Flyway CLI.

2. After downloading and extracting the files, copy the `flyway-11.3.1` folder to `C:\Flyway`. Then, add `C:\Flyway\flyway-11.3.1` to your PATH environment variable.

3. You can then open Command Prompt and enter `flyway --version` to confirm that it is able to be invoked.

### Install DBeaver

1. Download [DBeaver Community](https://dbeaver.io/download/).
    - I used the default installation options. The only thing I changed was to install for all users instead of the current user.

2. Launch DBeaver.
    - The first time you launch, you will be asked if you want to create a sample database. Select No.

3. Go to Database -> New Database Connection. Select PostgreSQL and then click Next.

4. On the Connection Settings page, leave all options as default and click Finish. You should now see an entry "postgres" in the Database Navigator panel.

5. Click the expand arrow next to the postgres database connection. This will prompt you to download some drivers. Click the Download button.
    - Note that you may get an error message about the database connection after the drivers are downloaded. You can ignore this.

6. Now that the drivers are downloaded, you can right click on the postgres database connection and delete it. We just wanted to use this connection as a way to get the required drivers downloaded.

## Database Setup

Now that we have the required software, we can create the databases and database user that will be used for the coding exercise.

### Create Database User

In command prompt, run the following command:

```
psql -c "create user milk with password 'milk';" -U postgres
```

In this command, we are using the superuser "postgres" to create a new database user "milk". 
You will be prompted for a password. Provide the password that you entered in Step 1 of the [Install PostgreSQL](#install-postgresql) section.
You will need to enter this password any time you run a command that have includes the `-U postgres` parameter.

### Create Databases

Run each of the four commands below, one at a time. This will create two a test and development database and grant the milk user ownership over both.

```
psql -c "create database milk_test;" -U postgres
psql -c "alter database milk_test owner to milk;" -U postgres
psql -c "create database milk_development;" -U postgres
psql -c "alter database milk_development owner to milk;" -U postgres
```

### Run Database Migrations

1. Open Command Prompt and navigate to your project directory, for example: `cd C:\Users\rmatzke1\Workspace\the-milk-problem`

2. Enter `set FLYWAY_CLEAN_DISABLED=false`. This will set an environment variable that is needed for the Flyway commands below.
    - Note that this environment variable does not persist to other terminal sessions. You will need to set this variable any time you open a new terminal if you are going to run any Flyway commands.

3. Enter this command to run the migration for the test database:

    ```
    flyway -user=milk -password=milk -url="jdbc:postgresql://localhost:5432/milk_test" -locations=filesystem:databases/milk clean migrate
    ```

4. Enter this command to run the migration for the development database:

    ```
    flyway -user=milk -password=milk -url="jdbc:postgresql://localhost:5432/milk_development" -locations=filesystem:databases/milk clean migrate
    ```

### Populate Database

1. Open Command Prompt and navigate to your project directory, for example: `cd C:\Users\rmatzke1\Workspace\the-milk-problem`

2. Setup environment variables by running this command:

    ```
    set JDBC_DATABASE_URL=jdbc:postgresql://localhost:5432/milk_development & set JDBC_DATABASE_USERNAME=milk & set JDBC_DATABASE_PASSWORD=milk
    ```

3. Run the command below to populate the development database with some data. Note that this is using the milk user, not the postgres user. So, when prompted for a password you will need to enter "milk".

    ```
    psql -f applications/products-server/src/test/resources/scenarios/products.sql milk_development -U milk
    ```

### Setup Database Connections

Now that the databases are setup and populated, you can use DBeaver to look at their structure and data.

1. Open DBeaver and go to Database -> New Database Connection.

2. Select PostgreSQL and click Next.

3. Set the following values. Leave the rest as their default.

    - Database: milk_test
    - Username: milk
    - Password: milk

4. Click Finish.

5. Repeat the steps above for the milk_development database.

You should now see two database connections in the Database Navigator panel. You can expand these until you see the tables. 

For example: milk_test -> Databases -> milk_test -> Schemas -> public -> tables

You can right click on the products table and select "View Data" to see what data has been inserted into the table.

## Final Notes

The only other thing I want to add is with regards sourcing the `.env` file. If you switch to using the IntelliJ IDEA integrated terminal (which you'll probably do for running the tests), you will need to set some environment variables again.
- To do so, you should be able to just run the same command from Step 2 in the [Populate Database](#populate-database) section of this guide.
- Remember that these environment variables do not persist between terminal sessions. So, if you open multiple sessions (one for server, one for client, one for tests), you'll probably need to set the variables in all of them.

That's pretty much all. You should now be able to work on the exercise. As the [assignment document](https://colorado.initialcapacity.io/contents/one-phase-commit) says, look for _todo_ items in the codebase to get started.
