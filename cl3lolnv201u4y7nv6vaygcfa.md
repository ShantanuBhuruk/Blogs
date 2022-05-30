## REST API Using Falcon and SQLObject

In this post, we will create a rest API for our application using falcon. This application will perform following tasks:

- Create a Student (POST request)
- Get a Student (GET request)
- Edit a Student (PUT request)
- Delete a Student (DELETE request)

We will use SQLObject as our resource provider for the APIs and gunicorn to serve those APIs.

> Note: For demonstration of the implementation of the topic of this post I have used Ubuntu 20.04.4 LTS. So all the installations are done using ubuntu commands.

### Prerequisites
Before starting the REST API development Using Falcon you should have knowledge of Python, REST and MySql. You should have python3, MySql server installed on your machine.

### API Flow
An API application consists of multiple layers. The code is divided in multiple layers to reduce the code complexity, test-ability and to improve the code readability. The core functionality of a API application is to  serve the requests coming from various clients and provide them a proper response. This resides in a service layer. A request from a client first reaches to out application routes and the it is forwarded to the appropriate Request Handlers i.e. Service layer. The service layer receives a request Validates it and then extracts out the data from the request and call the repository. Repository is basically a intermediate  layer between the Services and Models. All the database queries and the data transformation is done in repository. Repository then calls Models to perform the database operations. Models are just classes that represent a database table.

In this post I will start building the application from the lowest layer i.e model layer moving towards the topmost layer i.e. routes layer. The reason behind this is while developing an application its more easier to understand the code when you start it from the lower layer adding layers on top of it. But the actual flow begins from routes navigating towards the models via services and repositories.


### Install Falcon, SQLObject, gunicorn

Set up and activate a virtual environment using python3. Install Falcon, SQLObject and gunicorn.



```
pip install falcon
pip install SQLObject
pip install gunicorn
``` 

> Note : For this post I am using MySql as Database server. So to connect with the database we need to install mysql client and mysql-connector-python with pip but before that we need to install build-essentials which are mandatory to install mysqlclient

To connect with MySql database we need to install mysqlclient and mysql-connector-python by executing following commands:

```
sudo apt update
sudo apt install -y build-essential
sudo apt install default-libmysqlclient-dev -y
pip install mysqlclient
pip install mysql-connector-python
``` 

### Create database Connection
Once the package installation is done, we will set up database connection which will be used by SQLObject to perform database operations.

Create a Connection.py file with following code:

```
from sqlobject.mysql import builder

MySQLConnection = builder()
conn = MySQLConnection(user="root", password="password", host="127.0.0.1", db="College")

``` 
> Note: Here I have used my database configurations (database url, database name and credentials). Please replace those with your database configurations.

Here we just imported a builder from sqlobject.mysql and established a database connection using our database configurations and stored it inside a variable so that it can be accessed later.

### Models
Now we will create a Student Model mapped with Student table in our database.
Create StudentModel.py file with following code:

```
from sqlobject import *
from Connection import conn


class Student(SQLObject):
    _connection = conn
    name = StringCol(length=32, notNone=True)
    emailid = StringCol(length=32, notNone=True)
    rollno = IntCol(notNone=True)
    age = IntCol()

    def get_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "emailid": self.emailid,
            "rollno": self.rollno,
            "age": self.age
        }


Student.createTable(ifNotExists=True)
``` 
Here we have created a Student Model representing Student table. All the attributes defined in student class are columns in Student table. We have also passed a connection object which we created earlier so that the models can communicate with database.

### Define Repository
In order to perform database operations consuming the Models we will create the repository which will act as an middle layer between our API and our Models.
Create StudentRepository.py file with following code:

```
from sqlobject import *
import falcon
from StudentModel import Student


class StudentRepository:
    def get_student(self, student_id=None, name=""):
        if student_id is None:
            return Student.select(LIKE(Student.q.name, "%" + name + "%"))
        else:
            try:
                return Student.get(student_id)
            except SQLObjectNotFound:
                raise falcon.HTTPBadRequest(title='Wrong Student id',
                                            description='Please provide valid author id to get info')

    def add_student(self, student_data):
        try:
            student = Student(name=student_data["name"], emailid=student_data["emailid"], rollno=student_data["rollno"], age=student_data["age"])
            return student.get_dict()
        except Exception:
            raise falcon.HTTPBadRequest(title="Please provide valid data",
                                        description="The data you provided cannot be proccessed! Try again")

    def update_student(self, student_id=None, student_data={}):
        if student_id is not None:
            student = self.get_student(student_id=student_id)
            student_dict = student.get_dict()
            for k, v in student_data.items():
                student_dict[k] = v
            student.set(name=student_dict['name'], emailid=student_data["emailid"], rollno=student_data["rollno"], age=student_data["age"])
        else:
            raise falcon.HTTPBadRequest(title="Please provide valid data",
                                        description="The data you provided cannot be proccessed! Try again")

    def delete_student(self, student_id=None):
        if student_id is not None:
            Student.delete(student_id)
        else:
            raise falcon.HTTPBadRequest(title="Please provide valid data",
                                        description="The data you provided cannot be proccessed! Try again")

``` 

Here the repository is the file which actually performs the intermediary operations and calls models to perform database operations. It actually acts as a middle layer between our API Services and the Models. In repositories we write our custom queries and transform the data from request as well as from the database as per the requirements.

### Define Service (Request Handler)
Now we will define our Service which actually handles the Requests and returns responses to client.
Create StudentService.py file with following code:

```
import json
import falcon
from StudentRepository import StudentRepository

student_repository = StudentRepository()


class StudentService:
    def on_get(self, req, resp):
        params = req.params
        if params and params["name"]:
            student = student_repository.get_student(name=params["name"])
            resp.status = falcon.HTTP_200
            if student.count():
                resp.media = {
                    "Student": student.getOne().get_dict()
                }
            else:
                resp.media = {
                    "Student":{}
                }
        else:
            students = []
            for student in student_repository.get_student():
                students.append(student.get_dict())

            resp.status = falcon.HTTP_200
            resp.media = {
                "Students": students
            }

    def on_post(self, req, resp):
        req_data = json.loads(req.stream.read())
        student = student_repository.add_student(student_data=req_data)
        resp.status = falcon.HTTP_201
        resp.media = student

    def on_put(self, req, resp):
        req_data = json.loads(req.stream.read())
        params = req.params
        if params and params["student_id"]:
            student_id = params["student_id"]
            student_repository.update_student(student_id=student_id, student_data=req_data)
            resp.media = {'author': student_repository.get_student(student_id=student_id).get_dict()}
            resp.status = falcon.HTTP_200
        else:
            resp.status = falcon.HTTP_400
            resp.media = {
                "msg": "Please re-verify the request"
            }

    def on_delete(self, req, resp):
        params = req.params
        if params and params["student_id"]:
            student_id = params["student_id"]
            student_repository.delete_student(student_id=student_id)
            resp.status = falcon.HTTP_200
            resp.media = {
                "msg": "Record deleted...!!!"
            }
        else:
            resp.status = falcon.HTTP_400
            resp.media = {
                "msg": "Please re-verify the request"
            }
``` 
Here the service actually retrieves the data or parameters from the request and calls the repository to generate the response. Once the operation is completed the service generates a response and returns it to the caller. The service implements the HTTP methods like GET, POST, PUT, DELETE to map the HTTP Request. In our case we have implemented the on_get, on_put, on_post, on_delete methods.

### Set up the Application
After defining models and repositories and services, we add API  urls in app.py.

Create a app.py (you can name it anything you prefer) which will serve as an entry-point of our application. Following will be the code inside our app.py file

```
import falcon
from StudentService import StudentService

app = falcon.App()
app.add_route("/student", StudentService())
``` 

Here we just imported falcon framework and called the App method of falcon which will create a Falcon application. After creating a falcon application we have added the API route to the app which is our REST API end point. This API route is mapped to StudentService class which is actually going to handle all of our API requests and generate the responses.

Your project directory should look as below:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653489429871/CUb79ljpn.png align="left")

### Running the app
Run the server using following command.

```
gunicorn --reload app:app
``` 

Once the app is running we can call our REST API to perform CRUD Operations on the Student Table. Use Postman for trying out the APIs. Just try http://localhost:8000/student on your browser to get the list of students.
You can refer to [this](https://drive.google.com/file/d/1R2hWQqitNu3SM9KGkwvl3zl9g8kVSvL1/view?usp=sharing) postman collection for testing the APIs.

Yaay we have just built a REST API with Falcon and SQLObject.
![celebration.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1653475648373/7lQHuc_Ly.gif align="left")


### Further Readings
I will explain how to secure your REST APIs in Falcon using JWT in my blog post of this series ([here](https://shantanubhuruk.hashnode.dev/secure-your-falcon-rest-apis-with-jwt)).




