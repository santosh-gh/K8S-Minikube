# Deploy PostgreSQL Instance to Kubernetes
https://sweetcode.io/how-to-deploy-postgresql-instance-to-kubernetes/
https://adamtheautomator.com/postgres-to-kubernetes/



Kubernetes is an open-source technology for automating container orchestration deployment, scaling, and management. As a database engineer, avoiding setting up your databases locally can be fun. Infrastructures such as Kubernetes let you deploy and manage databases using container technologies. This guide will help you learn how to run a PostgreSQL database to Kubernetes.

Requirements
To follow along with this guide, you will need the following tools:

Docker Engine installed and running on your computer.
Kubectl for running Kubernetes command installed on your computer
A Kubernetes cluster management tool installed, such as Minikube. In this case, ensure Minikube is running using the command minikube start.
Why Run the PostgreSQL Database to Kubernetes
This approach allows you to run the PostgreSQL database as a service (DaaS). It will enable you to access and use a database without setting up or maintaining the infrastructure required to run it. This boosts database characteristics such as scaling, high availability, backups and recovery options. Running the PostgreSQL database in Kubernetes allows you to leverage the following benefits:

Running the PostgreSQL database in Kubernetes allows you to quickly scale on Kubernetes by adding more replicas (copies of the database) as needed. This allows scaling database usage-based traffic to ensure high availability.
Using Kubernetes to deploy PostgreSQL makes it easy to manage the lifecycle of your database. You can use Kubernetes to deploy, update, and manage the availability of your database.
Running PostgreSQL database in Kubernetes ensures the availability of your database by automatically restarting failed pods and containers running a PostgreSQL instance.
Deploying PostgreSQL to Kubernetes Cluster allows you to run your database in any cloud or on-premises environment. This makes your database portable and easy to migrate as needed.
Now, let’s dive and learn how to run a PostgreSQL database to Kubernetes. This guide will run a PostgreSQL using Kubernetes Deployment and Service Resources.

Deploying PostgreSQL Using Kubernetes
A deployment resource manages a set of replicas of pods. A pod refers is a group of containers deployed together on the same node. A Deployment ensures the desired number of replicas of the pod are running at any given time. It updates the pod and ensures rolling out new versions or back to previous ones.

A Service resource exposes a set of pods to other resources within or outside the cluster. A Service acts as a traffic distribution centre for the created pods. Let’s dive in and create a Deployment to manage a PostgreSQL database and a Service and expose it to other resources within and outside the cluster.

Creating the Volumes
A database involves data. You need to read and write data. Therefore, you need to persist this data so that an application can access and use it anytime. Thus, when deploying a database service to Kubernetes, a pod is set to a volume to persist your data. Go ahead and create a db-persistent-volume.yaml file. In this file, create a resource to set up Persistent Volume (PV) as follows:

apiVersion: v1
# Kind for volume chain
kind: PersistentVolume
metadata:
  # Name the persistent chain
  name: postgresdb-persistent-volume
  # Labels for identifying PV
  labels:
    type: local
    app: postgresdb
spec:
  storageClassName: manual
  capacity:
    # PV Storage capacity
    storage: 8Gi
  # A db can write and read from volumes to multiple pods
  accessModes:
    - ReadWriteMany
  # Specify the path to persistent the volumes  
  hostPath:
    path: "/data/db"
To allow a cluster to request access to the data storage, you need to set up a PersistentVolumeClaim (PVC) to the corresponding Persistent Volume. PVC is bound to a PV. It ensures the pods created within the designated cluster can access a volume to store data.

To create PVC, add a new file and name it db-volume-claim.yaml. In this file create PVC as follows:

apiVersion: v1
# define a resource for volume chain
kind: PersistentVolumeClaim
metadata:
  # Name the volume chain
  name: db-persistent-volume-claim
spec:
  storageClassName: manual
  accessModes:
    # Allow ReadWrite to multiple pods
    - ReadWriteMany
  # PVC requesting resources
  resources:
    requests:
      # the PVC storage
      storage: 8Gi
Creating Database Secrets
Databases require access control using passwords and users. You can use Kubernetes to create these credentials. To do this, use ConfigMap to create environment variables to store the database configurations.

You need to create a db-configmap.yaml file:

apiVersion: v1
# Kind for kubernets ConfigMap
kind: ConfigMap
metadata:
  # Name your ConfigMap
  name: db-secret-credentials
  labels:
    app: postgresdb
data:
  # User DB
  POSTGRES_DB: testDB
  # Db user
  POSTGRES_USER: testUser
  # Db password
  POSTGRES_PASSWORD: testPassword
Creating the Deployment Resource
Deployment resource creates and manages a set of replicas of a pod to run an application on Kubernetes. On top of that, it specifies all the needed artifacts to execute the application. This includes your application image and ports to expose it.

Additionally, it also allows you to add volumes to the pods on the cluster. In this example, we have specified the volumes that PostgreSQL requires. PersistentVolumeClaim must be specified during deployment to ensure the cluster can access the storage and store data. Volumes must be attached to the pods.

Create a db-deployment.yaml and build a PostgreSQL deployment manifest as follows:

# Kubernetes API version
apiVersion: apps/v1
# Deployment object
kind: Deployment
metadata:
  # The name of the Deployment
  name: postgresdb
spec:
  # Replicas for this Deployment
  replicas: 3
  selector:
    # labels the pods
    matchLabels:
      app: postgresdb
  template:
    metadata:
      labels:
        # The label the pods created from the pod template should have
        app: postgresdb
    spec:
      containers:
        # The container name to execute pods
        - name: postgresdb
          # pull postgresimage from docker hub
          image: postgres
          ports:
            # Assign ports to expose container
            - containerPort: 5432
          envFrom:
            # Load the environment variables/PostgresSQL credentials
            - configMapRef:
                # This should be the ConfigMap name created ealier
                name: db-secret-credentials
          volumeMounts:
            # The volume mounts  for the container
            - mountPath: /var/lib/postgres/data
              name: db-data
      # Volumes attached to the pod
      volumes:
        - name: db-data
          persistentVolumeClaim:
            # reference the PersistentVolumeClaim
            claimName: db-persistent-volume-claim
Creating the Service Resource
A service exposes a deployment. It provides a stable network endpoint for accessing a cluster. To create a service resource, add a db-service.yaml file and include the following:

apiVersion: v1
# Kind for service
kind: Service
metadata:
  # Name your service
  name: postgresdb
  labels:
    app: postgresdb
spec:
  # Choose how to expose your service
  type: NodePort
  ports:
    # The port number to expose the service
    - port: 5432
  # Pod to route service traffic  
  selector:
    app: postgresdb
Deploying PostgreSQL
You now have the needed resources. It’s time to run them on Kubernetes. First, you need to deploy the volumes before creating the pods. Below are the commands required to do so:

Deploy the PV:
kubectl apply -f db-persistent-volume.yaml
Deploy the PVC
kubectl apply -f db-volume-claim.yaml
The environment variables are needed by the cluster. Deploy them as follows:
kubectl apply -f db-configmap.yaml
Next, create the deployment and add pods replicas.
kubectl apply -f db-deployment.yaml
Finaly run the service to expose the cluster
kubectl apply -f db-service.yaml
Testing the Deployment
To check if the deployed pod is running, execute the following command:

kubectl get all


Consequently, you can run the following command to get a visual of all the workloads you have just added:

minikube dashboard


To check the database deployment, run the following command:

kubectl exec -it postgresdb-5b9bf77c46-6xhx2 -- psql -h localhost -U testUser --password -p 5432 testDB
Note that:

postgresdb-5b9bf77c46-6xhx2 is a pod name that is running your database. Copy the name as such.
testUser is the database user created using ConfigMap.
testDB is the database name created using ConfigMap.
The above command will allow you to access the testDB created by the ConfigMap.
Go ahead and run this command based on your specification.



This will allow you to add the password that you added to your ConfigMap. In this case, POSTGRES_PASSWORD is testPassword.



And there you have your deployed PostgreSQL to Kubernetes cluster.