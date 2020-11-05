# crud
#NOTE: I am not able to varify the working of ingress as i am working with docker edge and not minikube.


#Deployment Steps

#Clone the repository
git clone https://github.com/jayeshanuj/crud.git

#go to crud directory
cd crud

#provide executable permissions to deploy.sh
chmod 755 deploy.sh

#Run the script deploy.sh. It will apply the k8s manifest files.
./deploy.sh

#Check that all the k8s objects are deployed correctly.
kubectl get all;kubectl get secrets;kubectl get pv;kubectl get pvc.

#Enable the ingress addon
minikube addons enable ingress



#Next, you need to update your /etc/hosts file to route requests from the host we defined, hello.world, to the Minikube instance.
#Add an entry to /etc/hosts:

$ echo "$(minikube ip) hello.world" | sudo tee -a /etc/hosts

---------------------------------------------------------------------------


# Post steps

#Run below command to check that the books DB does not exist.
POD_NAME=$(kubectl get pod -l service=postgres -o jsonpath="{.items[0].metadata.name}")

$ kubectl exec $POD_NAME --stdin --tty -- psql -U sample

psql (12.1)
Type "help" for help.

sample=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 books     | sample   | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 sample    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(5 rows)

sample=#




# run below command to create a books database. Note that this is being done in postgres DB pod.
POD_NAME=$(kubectl get pod -l service=postgres -o jsonpath="{.items[0].metadata.name}")
kubectl exec $POD_NAME --stdin --tty -- createdb -U sample books

# Run this command to verify that the books DB is created. At this point in time DB installation is varified.
$ kubectl exec $POD_NAME --stdin --tty -- psql -U sample

psql (12.1)
Type "help" for help.

sample=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 books     | sample   | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 sample    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(5 rows)

sample=#




#apply the migrations, and seed the database. Note that this is being done in flask pod.

FLASK_POD_NAME=$(kubectl get pod -l app=flask -o jsonpath="{.items[0].metadata.name}")
kubectl exec $FLASK_POD_NAME --stdin --tty -- python manage.py recreate_db
kubectl exec $FLASK_POD_NAME --stdin --tty -- python manage.py seed_db



#Run below command to verify that books db is populated with data by flask application.At this point in time flask installation is varified and also the connectivity between #Flask front end and postgres backend is varified.
$ kubectl exec $POD_NAME --stdin --tty -- psql -U sample

psql (12.1)
Type "help" for help.

sample=# \c books
You are now connected to database "books" as user "sample".
books=# select * from books;

 id |                  title                   |    author     | read
----+------------------------------------------+---------------+------
  1 | On the Road                              | Jack Kerouac  | t
  2 | Harry Potter and the Philosopher's Stone | J. K. Rowling | f
  3 | Green Eggs and Ham                       | Dr. Seuss     | t
(3 rows)

-------------------------------------------------------------------------------
