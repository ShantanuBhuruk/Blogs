## REST API Using Falcon Framework

There are 2 kinds of frameworks, full-featured and bare frameworks. If you would rather develop a RESTFul API for your projects with Python, in general, you consider structure the following.

1) Flask with Flask-RESTFul

2) Django + REST Framework

On the other hand, there is a very good light-weight API framework available in Python called Falcon.

Accordingly, Falcon claims (I agree with them),
>other frameworks weigh you down with tons of dependencies and unnecessary abstractions. Falcon cuts to the chase with a clean design that embraces HTTP and the REST architectural style.

[Here](https://falcon.readthedocs.io/en/stable/) is a official documentation of Falcon Framework.

In this post, we will create a rest API for our application using falcon. This application will perform following tasks:

- Create a Student (POST request)
- Get a Student (GET request)
- Edit a Student (PUT request)
- Delete a Student (DELETE request)

We will use native methods to perform database operations for the APIs and gunicorn to serve those APIs.

### Prerequisites
Before starting the REST API development Using Falcon you should have knowledge of Python, REST and MySql. You should have python3, MySql server installed on your machine.

### Install Falcon and gunicorn

Set up and activate a virtual environment using python3. Install Falcon and gunicorn


```
pip install falcon
pip install gunicorn
``` 

### Setup Falcon Application

Once the package installation is done, we will set up our Falcon Application.
First of all we will create a class which will handle all the request and generate responses after performing the Database Operations. So create  StudentDBOperator.py file which will have below code to handle all GET, POST, PUT and DELETE requests and perform the database operations with native methods.


```
"""
Basic Database CRUD Operations using MySql with Python
Written by Shantanu Bhuruk
"""
import json
import falcon
import mysql.connector

"""
Student Database Operations Class
"""


class StudentDBOperator:
    con = None
    cur = None

    __json_content = {}

    """
    Create a database connection
    """

    def __init__(self):
        pass

    def __validate_json(self, req):
        try:
            self.__json_content = json.loads(req.stream.read())
            print("Valid Input JSON")
            return True
        except ValueError as e:
            self.__json_content = {}
            print("Invalid Input JSON")
            return False

    def create_connection(self):
        self.con = mysql.connector.connect(host="localhost", user="root", passwd="")
        print(f"Connected to Database Server {self.con.server_host}:{self.con.server_port}")
        return self.con

    """
    Close Database Connection
    """

    def close_connection(self):
        self.con.close()

    """
    Create a cursor
    """

    def create_cursor(self):
        self.cur = self.con.cursor()
        self.cur.execute("Use Test")
        print("Using Database Test...!!!")
        return self.cur

    """
    Insert Student into the Database
    """

    def on_post(self, req, resp):
        self.create_connection()
        self.create_cursor()
        resp.status = falcon.HTTP_201
        validated = self.__validate_json(req)
        output = {}
        if validated:
            insert_sql = "INSERT INTO STUDENT (NAME, BRANCH, ROLL, SECTION, AGE) VALUES (%s, %s, %s, %s, %s)"
            req_params = self.__json_content
            values = (req_params["name"], req_params["branch"], req_params["roll_no"], req_params["section"], req_params["age"])
            self.cur.execute(insert_sql, values)
            self.con.commit()
            print(f"Record Inserted: {values}")
            output = {
                "msg": "Record Inserted {}".format(values)
            }
        else:
            output = {
                "msg": "Invalid Json Input"
            }
        self.close_connection()
        resp.media = output

    """
    Update Student
    """

    def on_put(self, req, resp):
        self.create_connection()
        self.create_cursor()
        resp.status = falcon.HTTP_200
        update_sql = "UPDATE STUDENT SET NAME = %s, BRANCH = %s, SECTION = %s, AGE = %s WHERE ROLL = %s"
        req_params = json.loads(req.stream.read())
        values = (req_params["name"], req_params["branch"], req_params["section"], req_params["age"], req_params["roll_no"])
        self.cur.execute(update_sql, values)
        self.con.commit()
        print(f"Record Updated: {values}")
        output = {
            "msg": "Record Updated {}".format(values)
        }
        self.close_connection()
        resp.media = output

    """
    Delete Student
    """

    def on_delete(self, req, resp):
        self.create_connection()
        self.create_cursor()
        resp.status = falcon.HTTP_200
        delete_sql = "DELETE FROM STUDENT WHERE ROLL = %(a)s"
        req_params = json.loads(req.stream.read());
        self.cur.execute(delete_sql, {"a": req_params["roll_no"]})
        self.con.commit()
        print(f"Record with Roll No: {req_params['roll_no']} has been deleted")
        output = {
            "msg": "Record with Roll No: {} has been deleted".format(req_params['roll_no'])
        }
        self.close_connection()
        resp.media = output

    """
    Select all Students from Database
    """

    def on_get(self, req, resp):
        self.create_connection()
        self.create_cursor()
        resp.status = falcon.HTTP_200
        params = req.params
        if params:
            self.cur.execute("SELECT * FROM STUDENT WHERE NAME = %(a)s", {"a": params["name"]})
        else:
            self.cur.execute("SELECT * FROM STUDENT")
        records = self.cur.fetchall()

        print("Fetched data from Student : ")
        students = []
        for record in records:
            print(record)
            student = dict()
            student["Name"] = record[0]
            student["Branch"] = record[1]
            student["Roll No"] = record[2]
            student["Section"] = record[3]
            student["Age"] = record[4]
            students.append(student)

        output = {
            "students": students
        }
        self.close_connection()
        resp.media = output


if __name__ == "__main__": 
    pass

``` 



Now, Create an app.py file (you can name it anything you prefer) which will serve as an entry-point of our application. 



![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653461316086/pFCT4q22A.png align="left")

Following will be the code inside our app.py file 


```
import falcon
from StudentDbOperator import StudentDBOperator

app = falcon.App()
app.add_route("/student", StudentDBOperator())

``` 

Here we just imported falcon framework and called the App method of falcon which will create a Falcon application. After creating a falcon application we have added the API route to the app which is our REST API end point. This API route is mapped to **StudentDBOperator** class which is actually going to handle all of our API requests and generate the responses.

### Running the app

Run the server

```
gunicorn --reload app:app

``` 

Once the app is running we can call our REST API to perform CRUD Operations on out Student Table.
Use Postman for trying out the APIs.
Just try http://localhost:8000/student to get the list of students

Yaay we have just built a REST API with Falcon.
![celebration.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1653463101003/rZQuU5dQ1.gif align="left")

### Further Readings
I will explain Falcon REST API Implementation with SQLObject in my another post [here](https://shantanubhuruk.hashnode.dev/rest-api-using-falcon-and-sqlobject) where SQLObject will act as our resource provider for the APIs. 
