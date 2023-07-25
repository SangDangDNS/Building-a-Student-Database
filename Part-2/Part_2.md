## Instructions
For this Part 2, you will complete your student database from Part 1 while diving deeper into SQL commands.  

You can use the `students.sql` file you created at the end of Part 1 to rebuild it. 

You will use docker compose to create a container docker for Postgres.

Pls create a file `docker-compose.yaml` and a folder `students_data`.    

File `docker-compose.yaml`  
```
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=students
    volumes:
      - "./students_data:/var/lib/postgresql/data:rw"
    ports:
      - "5432:5432"
```

To start a postgres instance, run this command:  
`sudo docker-compose up -d`

**Note:** If you want to stop that docker compose, pls enter this command: `sudo docker-compose down`  

You can rebuild the database by entering in a terminal where the .sql file is.  

```
psql -h <hostname> -p <port> -U <username> -d <database> < students.sql
```  

Exp: `psql -h localhost -p 5432 -U root -d students < students.sql`

Ensure that the .pgpass file is properly set up to avoid any password prompts. If the .pgpass file doesn't exist, create it in your home directory and set the appropriate permissions:

```
touch ~/.pgpass
chmod 600 ~/.pgpass
```

Open the .pgpass file in a text editor and add the following line with the appropriate values for your PostgreSQL server:

```
localhost:5432:students:root:your_password_here
``` 

To log in to PostgreSQL with psql to create your database. Do that by entering this command in your terminal:

```
psql -h <hostname> -p <port> -U <username> -d <database>
```

And then, you will create the Bash script file `student_info.sh`. Ensure the script has execution permission: 

```
chmod +x student_info.sh
```

File `student_info.sh`  

```
#!/bin/bash

# Set PGPASSFILE environment variable to point to the .pgpass file
export PGPASSFILE=/home/sang/.pgpass

# Info about my computer science students from students database

echo -e "\n~~ My Computer Science Students ~~\n"

PSQL="psql -h localhost -p 5432 -U root -d students --no-align --tuples-only -c"

echo -e "\nFirst name, last name, and GPA of students with a 4.0 GPA:"
echo "$($PSQL "SELECT first_name, last_name, gpa FROM students WHERE gpa = 4.0")"

echo -e "\nAll course names whose first letter is before 'D' in the alphabet:"
echo "$($PSQL "SELECT course FROM courses WHERE course < 'D'")"

echo -e "\nFirst name, last name, and GPA of students whose last name begins with an 'R' or after and have a GPA greater than 3.8 or less than 2.0:"
echo "$($PSQL "SELECT first_name, last_name, gpa FROM students WHERE last_name >= 'R' AND (gpa > 3.8 OR gpa < 2.0)")"

echo -e "\nLast name of students whose last name contains a case insensitive 'sa' or have an 'r' as the second to last letter:"
echo "$($PSQL "SELECT last_name FROM students WHERE last_name ILIKE '%sa%' OR last_name ILIKE '%r_'")"

echo -e "\nFirst name, last name, and GPA of students who have not selected a major and either their first name begins with 'D' or they have a GPA greater than 3.0:"
echo "$($PSQL "SELECT first_name, last_name, gpa FROM students WHERE major_id IS NULL AND (first_name LIKE 'D%' OR gpa > 3.0)")"

echo -e "\nCourse name of the first five courses, in reverse alphabetical order, that have an 'e' as the second letter or end with an 's':"
echo "$($PSQL "SELECT course FROM courses WHERE course LIKE '_e%' OR course LIKE '%s' ORDER BY course DESC LIMIT 5")"

echo -e "\nAverage GPA of all students rounded to two decimal places:"
echo "$($PSQL "SELECT ROUND(AVG(gpa), 2) FROM students")"

echo -e "\nMajor ID, total number of students in a column named 'number_of_students', and average GPA rounded to two decimal places in a column name 'average_gpa', for each major ID in the students table having a student count greater than 1:"
echo "$($PSQL "SELECT major_id, COUNT(*) AS number_of_students, ROUND(AVG(gpa), 2) AS average_gpa FROM students GROUP BY major_id HAVING COUNT(*) > 1")"

echo -e "\nList of majors, in alphabetical order, that either no student is taking or has a student whose first name contains a case insensitive 'ma':"
echo "$($PSQL "SELECT major FROM students FULL JOIN majors ON students.major_id = majors.major_id WHERE major IS NOT NULL AND (student_id IS NULL OR first_name ILIKE '%ma%') ORDER BY major")"

echo -e "\nList of unique courses, in reverse alphabetical order, that no student or 'Obie Hilpert' is taking:"
echo "$($PSQL "SELECT DISTINCT(course) FROM students FULL JOIN majors USING(major_id) FULL JOIN majors_courses USING(major_id) FULL JOIN courses USING(course_id) WHERE student_id IS NULL OR (first_name = 'Obie' AND last_name = 'Hilpert') ORDER BY course DESC")"

echo -e "\nList of courses, in alphabetical order, with only one student enrolled:"
echo "$($PSQL "SELECT course FROM students INNER JOIN majors_courses USING(major_id) INNER JOIN courses USING(course_id) GROUP BY course HAVING COUNT(student_id) = 1 ORDER BY course")"
```

Next, run it in your terminal with this command:

```
./student_info.sh
```

You should see this result:

```
$ ./student_info.sh 

~~ My Computer Science Students ~~


First name, last name, and GPA of students with a 4.0 GPA:
Casares|Hijo|4.0
Vanya|Hassanah|4.0
Dejon|Howell|4.0

All course names whose first letter is before 'D' in the alphabet:
Computer Networks
Computer Systems
Artificial Intelligence
Calculus
Algorithms

First name, last name, and GPA of students whose last name begins with an 'R' or after and have a GPA greater than 3.8 or less than 2.0:
Efren|Reilly|3.9
Mariana|Russel|1.8
Mehdi|Vandenberghe|1.9

Last name of students whose last name contains a case insensitive 'sa' or have an 'r' as the second to last letter:
Gilbert
Savage
Saunders
Hilpert
Hassanah

First name, last name, and GPA of students who have not selected a major and either their first name begins with 'D' or they have a GPA greater than 3.0:
Noe|Savage|3.6
Danh|Nhung|2.4
Hugo|Duran|3.8

Course name of the first five courses, in reverse alphabetical order, that have an 'e' as the second letter or end with an 's':
Web Programming
Web Applications
Server Administration
Network Security
Data Structures and Algorithms

Average GPA of all students rounded to two decimal places:
3.09

Major ID, total number of students in a column named 'number_of_students', and average GPA rounded to two decimal places in a column name 'average_gpa', for each major ID in the students table having a student count greater than 1:
|8|2.97
41|6|3.53
38|4|2.73
36|6|2.92
37|6|3.38

List of majors, in alphabetical order, that either no student is taking or has a student whose first name contains a case insensitive 'ma':
Computer Programming
Database Administration
Network Engineering
Web Development

List of unique courses, in reverse alphabetical order, that no student or 'Obie Hilpert' is taking:
Web Programming
Web Applications
Python
Object-Oriented Programming
Network Security
Data Structures and Algorithms
Computer Systems
Computer Networks
Algorithms

List of courses, in alphabetical order, with only one student enrolled:
Computer Networks
Computer Systems
Server Administration
UNIX
```