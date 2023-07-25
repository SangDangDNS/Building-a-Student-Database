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

Ensure that the .pgpass file is properly set up to avoid any password prompts. If the .pgpass file doesn't exist, create it in your home directory and set the appropriate permissions:

```
touch ~/.pgpass
chmod 600 ~/.pgpass
```

Open the .pgpass file in a text editor and add the following line with the appropriate values for your PostgreSQL server:

```
localhost:5432:students:root:your_password_here
``` 

You need to log in to PostgreSQL with psql to create your database. Do that by entering this command in your terminal:

```
psql -h <hostname> -p <port> -U <username> -d <database>
```

Create 4 tables for DB like the below:

```
students=# \d students
                                         Table "public.students"
   Column   |         Type          | Collation | Nullable |                   Default                    
------------+-----------------------+-----------+----------+----------------------------------------------
 student_id | integer               |           | not null | nextval('students_student_id_seq'::regclass)
 first_name | character varying(50) |           | not null | 
 last_name  | character varying(50) |           | not null | 
 major_id   | integer               |           |          | 
 gpa        | numeric(2,1)          |           |          | 
Indexes:
    "students_pkey" PRIMARY KEY, btree (student_id)
Foreign-key constraints:
    "students_major_id_fkey" FOREIGN KEY (major_id) REFERENCES majors(major_id)
```

```
students=# \d majors
                                       Table "public.majors"
  Column  |         Type          | Collation | Nullable |                 Default                  
----------+-----------------------+-----------+----------+------------------------------------------
 major_id | integer               |           | not null | nextval('majors_major_id_seq'::regclass)
 major    | character varying(50) |           | not null | 
Indexes:
    "majors_pkey" PRIMARY KEY, btree (major_id)
Referenced by:
    TABLE "majors_courses" CONSTRAINT "majors_courses_major_id_fkey" FOREIGN KEY (major_id) REFERENCES majors(major_id)
    TABLE "students" CONSTRAINT "students_major_id_fkey" FOREIGN KEY (major_id) REFERENCES majors(major_id)
```

```
students=# \d courses
                                         Table "public.courses"
  Column   |          Type          | Collation | Nullable |                  Default                   
-----------+------------------------+-----------+----------+--------------------------------------------
 course_id | integer                |           | not null | nextval('courses_course_id_seq'::regclass)
 course    | character varying(100) |           | not null | 
Indexes:
    "courses_pkey" PRIMARY KEY, btree (course_id)
Referenced by:
    TABLE "majors_courses" CONSTRAINT "majors_courses_course_id_fkey" FOREIGN KEY (course_id) REFERENCES courses(course_id)
```

```
students=# \d majors_courses
            Table "public.majors_courses"
  Column   |  Type   | Collation | Nullable | Default 
-----------+---------+-----------+----------+---------
 major_id  | integer |           | not null | 
 course_id | integer |           | not null | 
Indexes:
    "majors_courses_pkey" PRIMARY KEY, btree (major_id, course_id)
Foreign-key constraints:
    "majors_courses_course_id_fkey" FOREIGN KEY (course_id) REFERENCES courses(course_id)
    "majors_courses_major_id_fkey" FOREIGN KEY (major_id) REFERENCES majors(major_id)
```

And then, you will create the Bash script file `insert_data.sh`. Ensure the script has execution permission: 

```
chmod +x insert_data.sh
```

File `insert_data.sh`  

```
#!/bin/bash

# Set PGPASSFILE environment variable to point to the .pgpass file
export PGPASSFILE=/home/sang/.pgpass

# Script to insert data from courses.csv and students.csv into students database

PSQL="psql -h localhost -p 5432 -U root -d students --no-align --tuples-only -c"
echo $($PSQL "TRUNCATE students, majors, courses, majors_courses")

cat courses.csv | while IFS="," read MAJOR COURSE
do
  if [[ $MAJOR != "major" ]]
  then
    # get major_id
    MAJOR_ID=$($PSQL "SELECT major_id FROM majors WHERE major='$MAJOR'")

    # if not found
    if [[ -z $MAJOR_ID ]]
    then
      # insert major
      INSERT_MAJOR_RESULT=$($PSQL "INSERT INTO majors(major) VALUES('$MAJOR')")
      if [[ $INSERT_MAJOR_RESULT == "INSERT 0 1" ]]
      then
        echo Inserted into majors, $MAJOR
      fi

      # get new major_id
      MAJOR_ID=$($PSQL "SELECT major_id FROM majors WHERE major='$MAJOR'")
    fi

    # get course_id
    COURSE_ID=$($PSQL "SELECT course_id FROM courses WHERE course='$COURSE'")

    # if not found
    if [[ -z $COURSE_ID ]]
    then
      # insert course
      INSERT_COURSE_RESULT=$($PSQL "INSERT INTO courses(course) VALUES('$COURSE')")
      if [[ $INSERT_COURSE_RESULT == "INSERT 0 1" ]]
      then
        echo Inserted into courses, $COURSE
      fi

      # get new course_id
      COURSE_ID=$($PSQL "SELECT course_id FROM courses WHERE course='$COURSE'")
    fi

    # insert into majors_courses
    INSERT_MAJORS_COURSES_RESULT=$($PSQL "INSERT INTO majors_courses(major_id, course_id) VALUES($MAJOR_ID, $COURSE_ID)")
    if [[ $INSERT_MAJORS_COURSES_RESULT == "INSERT 0 1" ]]
    then
      echo Inserted into majors_courses, $MAJOR : $COURSE
    fi
  fi
done

cat students.csv | while IFS="," read FIRST LAST MAJOR GPA
do
  if [[ $FIRST != "first_name" ]]
  then
    # get major_id
    MAJOR_ID=$($PSQL "SELECT major_id FROM majors WHERE major='$MAJOR'") 

    # if not found
    if [[ -z $MAJOR_ID ]]
    then
      # set to null
      MAJOR_ID=null
    fi

    # insert student
    INSERT_STUDENT_RESULT=$($PSQL "INSERT INTO students(first_name, last_name, major_id, gpa) VALUES('$FIRST', '$LAST', $MAJOR_ID, $GPA)")
    if [[ $INSERT_STUDENT_RESULT == "INSERT 0 1" ]]
    then
      echo Inserted into students, $FIRST $LAST
    fi
  fi
done
```

Next, run it in your terminal with this command:

```
./insert_data.sh
```

When completed, pls enter in the terminal to dump the database into a students.sql file. It will save all the commands needed to rebuild it. Take a quick look at the file when you are done. The file will be located where the command was entered.  

```
pg_dump --clean --create --inserts --username=root -h localhost students > students.sql
```  

You can rebuild the database by entering in a terminal where the .sql file is.  

```
psql -h <hostname> -p <port> -U <username> -d <database> < students.sql
```  

Exp: `psql -h localhost -p 5432 -U root -d students < students.sql`

