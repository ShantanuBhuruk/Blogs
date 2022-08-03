## Custom sorting in SQL

Sometimes you require to order your data in a particular way, and often the use of typical sort modifiers like ASC and DESC are not enough and I am not kind of surprised that you are here because this is the most common challenge a enterprise application developer faces. I can say this a well known problem when you need to represent a bit differently sorted data than its actual existence in the database table. 

> If I am not wrong most of the people don't know the solution.


I’m going to show a very simple way for this well know problem in my opinion (I’ve experienced that for 5 times at least ). In this article I am going to show how the **CASE** statement in **SQL** can help you order  your data exactly the way you want. One of the best methods for handling complex sorting is using CASE. The CASE statement is quite useful, as it lets you add if-else logic to your queries. We’ll use this logic SQL ORDER BY specific values.

Here is the syntax for the SQL CASE expression in ORDER BY clause:

```
SELECT * FROM table_name
ORDER BY CASE 
     WHEN column_field = "value1" THEN priority1
     WHEN column_field = "value2" THEN priority2
     WHEN column_field = "value3" THEN priority3
     .
     ELSE priorityn 
     END 
ASC
```

Lets take an example of the Banking industry or Reporting in investment institution - documents area. Such documents would always have a status. In our example we would be using such statuses:

- IN_PROGRESS
- PENDING_APPROVAL
- APPROVED
- PUBLISHED

Now what every database admin will tell you over design phase is instead of storing these four same Strings in millions of records in our database, could you please (this is an order in fact, not a question) map them into numbers so that the memory utilisation will decrease 99% for that column. So what we will do is we will map these strings to a separate number and we will use these numbers in table columns. Suppose following is our mapping :

- IN_PROGRESS             ->  1
- PENDING_APPROVAL ->  2
- APPROVED                  ->  3
- PUBLISHED                 ->  4


**All happy? **Not exactly. I would say in 99% of such cases that statuses are really relevant from the end user perspective. These statuses are core part of the application, you can’t rely on numbers as users don’t know your mapping. Of course you will map them through your enum. That’s nice. But what if they need to sort them or you need to implement lazy loading after such sorting? Even worse that sorting order is not the same as values defined on our enum. 

## Ohh No...! We are caught in trouble here. 
![tom-and-jerry-jerry-the-mouse.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1659546688327/NvyAhrdoJ.gif align="left")

Exactly in such scenarios the **SQL ORDER BY CASE** Statement can help us out. In such scenario the only way to achieve such functionality is usage of ORDER BY CASE statement. This is the order we need to define on our statuses to satisfy end users expectations:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659546915052/oVUKaU2jZ.png align="left")

Lets consider following sample data :

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659546974369/VUSWFqzIS.png align="left")

Now we when this data is represented on the UI this and sorted according to status then it should be displayed in following order of statuses (alphabetical order):

- Approved
- In Progress
- Pending approval
- Published

But if we sort it on status column it would get displayed in different order as follows:

- In Progress
- Pending approval
- Approved
- Published

So we need to sort this data with some custom sorting mechanism which we can do with the help of ORDER BY CASE statement. We will prepare our SQL statement to get the desired output. So following will be our SQL query using the ORDER BY CASE statement.

```
select * from report order by CASE
    when status = 1 then 'In progress'
    when status = 2 then 'Pending approval'
    when status = 3 then 'Approved'
    when status = 4 then 'Published'
END
```
Now when we execute this query we will get the required output. Here in our output the records with status **"Aproved"** will appear at top then records with status **"In Progress"** then records with status **"Pending approval"** and records with status **"Published"** will appear at the bottom.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659552155606/MQDhZ2b1H.png align="left")

## Yehhhhh........!!! 

 
![yay-icegif-8.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1659549210225/QWUUD11G2.gif align="left")

## We got it right....

The **ORDER BY CASE** statement is a simple solution to our well known complicated looking problem.
