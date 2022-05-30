## Secure your Falcon REST APIs with JWT

It’s not always easy to secure REST APIs. In this post, we’ll discuss how to do just that using [JSON web tokens (JWT)](https://jwt.io/introduction).

### Prerequisites
To get into the security implementation of Falcon REST APIs with JWT one should know about Falcon, REST and last but not the least the JSON web tokens (JWT).

### Introduction
That being said, due to the nature and mechanics behind REST APIs, securing them is not always straightforward. What happens after the user submits their credentials? How do you know they’ve correctly logged in on their subsequent requests? You can’t keep a state on your server-side to signal that. So what do you do?

In this article, I want to share with you one very powerful yet simple way you can achieve this: using JSON Web Tokens. We are going to implement JWT authentication in our Falcon API Application throughout this post. I will refer to the example of /student API from my previous post which you can find [here](https://shantanubhuruk.hashnode.dev/rest-api-using-falcon-and-sqlobject).

Any interaction with our secure API would start with a login request. It would look something like this:

 
```
POST /login
``` 

Payload:

```
{
    “Username”: “shantanu”
    “Password”: “password”
}
``` 

After validating the credentials, the system would return a new JSON Web Token. Which will include the below payload:

```
{
    "user_id": user.id
}
``` 
To keep things simple, we’ll use an **"HS256"** algorithm for encoding the data, meaning we’ll be using the same secret, both, on our client and our API. 

For the purposes of this example, our secret will be:
```
secret
```

### Authentication mechanism  
 To authenticate the users and to generate the JWT we will create a route **/login** which will accept the Username and Password and after validating these credentials with the database it will generate a JWT and returns that in response to the request.

### jwt module of python
For the encoding and decoding the JWT we will be using the jwt module of python. This module is included in python so you don't need to install it separately.

### Create User Model
For validating the user credentials we need to check that if the user with these credentials exists in database. For that we will create a User model first.

Create UserModel.py file with following code:

```
from sqlobject import *
from Connection import conn


class User(SQLObject):
    _connection = conn
    username = StringCol(length=30, notNone=True, unique=True)
    password = StringCol(length=30, notNone=True)


User.createTable(ifNotExists=True)
``` 
User Model has the attributes mapped with the columns in User table from database. We are passing the global database connection object to the model so that it can communicate with the database.

### Define a User Repository
Now we will define a user repository which will query the database. It will execute the select statement with Username and Password in the where clause. It will execute the query on the database with the help of the user model.

Create a UserModel.py file with following code:

```
from UserModel import User
from sqlobject import *


class UserRepository:
    @staticmethod
    def get_user(username, password):
        return User.select(AND(User.q.username == username, User.q.password == password))
``` 

### Create login service (Request Handler)

Now we will define a login service which will handle the login requests from client and will provide the JWT in response by calling the User Repository.

Create LoginService.py file with following code:

```
import falcon
import jwt
import json

from UserRepository import UserRepository


class LoginService:
    def __init__(self):
        pass

    def login(self, req, resp):
        req_params = json.loads(req.stream.read())
        print("Attempting Login")

        if not req_params or not req_params["Username"] or not req_params["Password"]:
            raise falcon.HTTPBadRequest("Bad Request", "Please enter valid Username and Password")
        else:
            print("authenticating with username: {} and password: {}".format(req_params["Username"], req_params["Password"]))
            self._authenticate(req_params["Username"], req_params["Password"], req, resp)

    def _authenticate(self, username, password, req, resp):
        if not username or not password:
            raise falcon.HTTPBadRequest("Bad Request", "Please enter valid Username and Password")
        else:
            print("Fetching User Form DB")
            users = UserRepository.get_user(username, password)
            print("User Info: {}".format(users))
            if users.count() > 0:
                user = users[0]
                payload = {
                    "user_id": user.id
                }
                secret = "secret"
                algo = "HS256"
                token = jwt.encode(payload=payload, key=secret, algorithm=algo)
                print(token)
                resp.media = {
                    "token": token.decode("utf-8")
                }
                resp.status = falcon.HTTP_200
            else:
                raise falcon.HTTPUnauthorized("Unauthorized", "Invalid Credentials")

    def on_post(self, req, resp):
        self.login(req, resp)


def main():
    pass


if __name__ == "__main__": main()
``` 
In login service we just imported falcon, jwt and json modules. We have also imported the USerRepository which will be used to validate the user credentials in the database.
We have implemented a on_post http method here which will handle the the POST requests from client and after validating the request it will return a jwt in response.
on_post method internally calls the login method which will extract the request payload and gets the Username and Password from it. After getting the Username and Password it calls the authnticate method which actually validates the user credentials with the database with the help of User Repository an generates a appropriate response. If the credentials are valid it will generate a jwt including the user_id in payload and returns that else if the credentials are invalid it will return HTTPUnauthorized request error. Here we are using **jwt.encode()** method to generate the jwt. We have passed payload i.e {user_id:"<user_id>"} in our case, key i.e. "secret" in our case, algorithm which we decided to use is "HS256". Using these this will generate the jwt which will look similar to the below example jwt token.


```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.J_RIIkoOLNXtd5IZcEwaBDGKGA3VnnYmuXnmhsmDEOs
``` 

At the same time, any further request sent by the client app will contain this same token, which in turn, will be validated by the server by re-signing it every time and comparing results with the signature portion of the token.

In a typical JWT request, you’ll pass the token as part of the authorization header on the client-side after the client must have logged in, like so: 
```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.J_RIIkoOLNXtd5IZcEwaBDGKGA3VnnYmuXnmhsmDEOs
```

### Define a middleware to authenticate the requests
Now we will define a middleware which will intercept each and every request coming to our application and check if the request contains the JWT in authorization header. It extract outs the JWT and check if its a valid one. If not a valid one then the request gets rejected and Unauthorized response will be sent. If the JWT is valid then request will be processed forward. It decodes the JWT and extract out the payload and also validates the user_id from the payload.

Create a AuthHandler.py file with following code:

```
import falcon
import jwt
import sqlobject.dberrors
from UserModel import User


class AuthHandler:
    def process_request(self, req, resp):
        if "/login" in req.path:
            return

        if req.get_header("Authorization"):
            auth_header = req.get_header("Authorization").split(" ")
            token = auth_header[1]
            if token:
                if not self._is_token_valid(token):
                    description = "The provided auth token is not valid.Please request a new token and try again."
                    raise falcon.HTTPUnauthorized("Unauthorized", description)
        else:
            description = "The provided auth token is not valid.Please request a new token and try again."
            raise falcon.HTTPUnauthorized("Unauthorized", description)

    def _is_token_valid(self, token):
        try:
            payload = jwt.decode(jwt=token, key="secret", algorithms="HS256")
            userid = payload["user_id"]
            user = User.get(payload['user_id'])
            print("Authenticated, User : {}".format(user))
            return True
        except (jwt.DecodeError, jwt.ExpiredSignatureError, sqlobject.dberrors.Error):
            return False
``` 
Here the AuthHandler implements the process_request method which will get executed once request is made. The process_request method will check the request path and if its not "/login" then it will check for Authorization header. If Authorization header is not present the request will be rejected with Uathorized error. Then it will extract out the JWT from the Authorization header and validate that and extracts out the information from it by using the  **jwt.decode()** method. If the JWT is valid then only the request will be processed further otherwise it will return an Unauthorized error.

### Register a Login route to our application
Import LoginService in our app.py.

```
from LoginService import LoginService

``` 
add a route "/login" in our app.


```
app.add_route("/login", LoginService())
``` 

### Register AuthHandler as middleware in our Applicaion
Now we will register our AuthHandler class as a middleware in out application by simply using the **add_middleware()** of falcon.App(). By doing this we will be making sure that each request coming to our application should be intercepted and is secure.

Import AuthHandler in your app.py.


```
from AuthHandler import AuthHandler
``` 

and Add following line to register AuthHandler as a middleware.

```
app.add_middleware(AuthHandler())
``` 

Your project directory should look as below:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653574147444/ou05ZtN3T.png align="left")

### Running the app

Run the server using following command.

```
gunicorn --reload app:app
``` 

Once the app is running we can call our REST APIs served on http://localhost:8000.
First of all we will call the "/login" API to get the JWT and using that JWT we will try to call the other APIs. We will add a Authorization header in our API requests as Bearer token as below.

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.J_RIIkoOLNXtd5IZcEwaBDGKGA3VnnYmuXnmhsmDEOs
``` 
Before adding a Authorization header just try calling the get Student API it should give an Unauthorized Error as below.

```
{
    "title": "Unauthorized",
    "description": "The provided auth token is not valid.Please request a new token and try again."
}
``` 

Now we will try by adding the Authorization Header it should give 200 OK response with some data in response.

You can refer to [this](https://drive.google.com/file/d/1R2hWQqitNu3SM9KGkwvl3zl9g8kVSvL1/view?usp=sharing) Postman collection for testing.

Yehhh we are done. We just secured our Falcon REST APIs with JWT.

![achieve-achievement.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1653575039379/dxz-Jh5kt.gif align="left")














