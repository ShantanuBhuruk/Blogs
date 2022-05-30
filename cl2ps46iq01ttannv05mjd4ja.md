## REST and Its Constraints

**REST means Representational State Transfer** which means every URL represents some objects. It transfers the state of a thing from client to server or server to client by representation. It means that the state is never stored at the server side. The data required to complete the request is sent in the request itself. 
**REST** is an architectural style that defines a set of constraints to create web services and neither a Tool, a Protocol, nor a Library. 

### What it means to be RESTful


**Web services which follow the REST architectural style are known as RESTful web services**. It allows requesting systems to access and manipulate web resources by using a uniform and predefined set of rules. These Web services offer their Web resources in a textual representation and allow them to be read and processed with a stateless protocol. The operations that a client can do are predefined and well known, generally based on the HTTP protocols.

RESTful APIs and software aren’t based on some technological or programming advancement. The idea behind REST is to impose certain rules upon the API so that you get more performance, scalability, simplicity, modifiability, visibility, portability, and reliability.

Those are a lot of benefits, but how do REST APIs achieve them, you may ask. Simple, by following six constraints. In fact, these rules are the most defining features of the RESTful architectural style.

### The six architectural constraints of REST APIs
1. Client-server architecture
2. Statelessness
3. Uniform Interface
4. Layered system
5. Cacheability
6. Code on Demand


**Client-Server architecture : **

An API’s job is to connect two pieces of software without limiting their own functionalities. This objective is one of the core restrictions of REST: the client (that makes requests) and the server (that gives responses) stay separate and independent.

When done properly, the client and server can update and evolve in different directions without having an impact on the quality of their data exchange. This is especially important in various cases where there are plenty of different clients a server has to cater to. Think about weather APIs — they have to send data to tons of different clients (all types of mobile devices are good examples) from a single database.

**Statelessness:**

It means that the necessary state to handle the request is contained within the request itself and server would not store anything related to the session. In REST, the client must include all information for the server to fulfil the request whether as a part of query params, headers or URI. Statelessness enables greater availability since the server does not have to maintain, update or communicate that session state. There is a drawback when the client need to send too much data to the server so it reduces the scope of network optimization and requires more bandwidth

An example of a non-stateless API would be if, during a session, only the first call has to contain the API key, which is then stored server-side. The following API calls depend on that first one since it provides the client’s credentials.

In the same case, a stateless API will ensure that each call contains the API key and the server expects to see proof of access each time.

Stateless APIs have the advantage that one bad or failed call doesn’t derail the ones that follow.

**Uniform Interface:**

It is a key constraint that differentiate between a REST API and Non-REST API. It suggests that there should be an uniform way of interacting with a given server irrespective of device or type of application (website, mobile app).
There are four guidelines principle of Uniform Interface are:


- 
Resource-Based: Individual resources are identified in requests. For example: API/users.

- 
Manipulation of Resources Through Representations: Client has representation of resource and it contains enough information to modify or delete the resource on the server, provided it has permission to do so. Example: Usually user get a user id when user request for a list of users and then use that id to delete or modify that particular user.

- 
Self-descriptive Messages: Each message includes enough information to describe how to process the message so that server can easily analyses the request.

- 
Hypermedia as the Engine of Application State (HATEOAS): It need to include links for each response so that client can discover other resources easily.

**Layered system:**

To keep the API easy to understand and scale, RESTful architecture dictates that the design is structured into layers that operate together.

With a clear hierarchy for these layers, executing a command means that each layer does its function and then sends the data to the next one. Connected layers communicate with each other, but not with every component of the program. This way, the overall security of the API is also improved.

If the scope of the API changes, layers can be added, modified, or taken out without compromising other components of the interface.

**Cacheability:**

It’s not uncommon for a stateless API’s requests to have large overhead. In some cases, that’s unavoidable, but for repeated requests that need the same data, caching said information can make a huge difference.

The concept is simple: the client has the option to locally store certain pieces of data for a predetermined period of time. When they make a request for that data, instead of the server sending it again, they use the stored version.

The result is simple: instead of the client sending several difficult or costly requests in a short span of time, they only have to do it once.

A well-managed caching partially or completely eliminates some client–server interactions, further improving availability and performance.

**Code on Demand:**

Unlike the other constraints we talked about up to this point, the last one is optional. The reason for making “code on demand” optional is simple: it can be a large security risk.

The concept is to allow code or applets to be sent through the API and used for the application. As you can imagine, unknown code from a shady source could do some damage, so this constraint is best left for internal APIs where you have less to fear from hackers and people with bad intentions. Another drawback is that the code has to be in the appropriate programming language for the application, which isn’t always the case.

The upside is that “code on demand” can help the client implement their own features on the go, with less work being necessary on the API or server. In essence, it permits the whole system to be much more scalable and agile.

### Rules of REST API

There are certain rules which should be kept in mind while creating REST API endpoints.

- REST is based on the resource or noun instead of action or verb based. It means that a URI of a REST API should always end with a noun. Example: /api/users is a good example, but /api?type=users is a bad example of creating a REST API.
- HTTP verbs are used to identify the action. Some of the HTTP verbs are – GET, PUT, POST, DELETE, UPDATE, PATCH.
- A web application should be organized into resources like users and then uses HTTP verbs like – GET, PUT, PATCH, POST, DELETE to modify those resources. And as a developer it should be clear that what needs to be done just by looking at the endpoint and HTTP method used.

    Here are few examples considering the operations on Users Entity:
    ![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650616515757/KVvTPz0XN.png)

- Always use plurals in URL to keep an API URI consistent throughout the application.
- Send a proper HTTP code to indicate a success or error status.

### When an API is RESTful API or REST Like API
![When-Is-an-API-Truly-REST-vs-REST-Like- (1).jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1650617334605/UqzPeMTbt.jpg)

As I already mentioned **REST** is an architectural style that defines a set of constraints to create web services and neither a Tool, a Protocol, nor a Library.  **So if we write the API which follows all the 5 mandatory constraints which are mentioned above then the API is Called as RESTful API and if it follows few of the constraints then its not RESTful API but we can call it REST Like API.**

### Final Thoughts

Its not like if you don't follow all the constraints of REST then your API's will not work or they are error prone but **if you are willing to get more performance, scalability, simplicity, modifiability, visibility, portability, and reliability then we should always follow all of these REST constraints. Thats the main idea behind the RESTful API implementation.**