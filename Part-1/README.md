## Instructions
For this project, you will create a Bash script that uses SQL to enter information about computer science students into PostgreSQL.  

You have two .csv (`students.csv` and `courses.csv`) files with info about your computer science students. You should take a look at them. The top row in each file has titles, and the rest are values for those titles. You will be adding all that info to a PostgreSQL database.    

You will use a docker image of postgres. See [postgres](https://hub.docker.com/_/postgres) on DockerHub for more information.

To start a postgres instance, run this command:

```
$ mkdir students_data
$ docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="students" \
  -v $(pwd)/students_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

You need to log in to PostgreSQL with psql to create your database. Do that by entering this command in your terminal:

```
psql -h <hostname> -p <port> -U <username> -d <database>
```

And then, you will create the schema for DB students and the Bash script file `insert_data.sh`. Ensure the script has execution permission: 

```
chmod +x insert_data.sh
```

Next, run it in your terminal with this command:

```
./insert_data.sh
```

When completed, pls enter:  

```
pg_dump --clean --create --inserts --username=root -h localhost students > students.sql

```  

in the terminal to dump the database into a students.sql file. It will save all the commands needed to rebuild it. Take a quick look at the file when you are done. The file will be located where the command was entered.  

You can rebuild the database by entering:  

```
psql -h <hostname> -p <port> -U <username> -d <database> < students.sql
```  
in a terminal where the .sql file is.

Exp: `psql -h localhost -p 5432 -U root -d students < students.sql`

