# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### Database setup
#### 1. AWS configure and create Kubernetes cluster

Connect your local environment with AWS
```bash
## please get the keys of your AWS accounts for below commands
aws configure
aws configure set aws_session-token <YOUR AWS SESSION TOKEN>
```

Create a Kubernetes cluster with the name `sa-cluster`. You can chanbge the cluster name and nodegroup as your choice.
```bash
eksctl create cluster --name sa-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
```

With your local kubernetes context so that you can use `kuberctl` to interact with the created cluster on AWS
```bash
aws eks --region us-east-1 update-kubeconfig --name sa-cluster
```

#### 2. Create a Postgres database
Set up the Postgres database from YAML deployment artifacts

1. Create YAML for PersitenceVolumeClaims

    Please refer to `deployment-local/pvc.yaml`


2. Create YAML for PersitenceVolume
    
    Please refer to `deployment-local/pv.yaml`

    <sup><sub>Make sure the `storageClassName` on both these files are same.</sub></sup>

3. Create a Postgres deployment
    
    Please refer to `deployment-local/postgresql-deployment.yaml`

    Some env variables for database need to be taken care: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`

4. Create service for Postgres deployment

    Please refer to `deployment-local/postgresql-service.yaml`

5. Apply these YAMLs to deploy them to cluster

    ```bash
    kubectl apply -f pvc.yaml
    kubectl apply -f pv.yaml
    kubectl apply -f postgresql-deployment.yaml
    kubectl apply -f postgresql-service.yaml
    ```

#### 3. Initialize for database

* Connecting Via Port Forwarding
```bash
kubectl port-forward svc/postgresql-service 5433:5432 &
```

* Import the data
```bash
export POSTGRES_PASSWORD=<The password that you set in deployment-local/postgresql-deployment.yaml>

 PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U $POSTGRES_USER -d $POSTGRES_DB -p 5432 <  db/1_create_tables.sql
 PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U $POSTGRES_USER -d $POSTGRES_DB -p 5432 <  db/2_seed_users.sql
 PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U $POSTGRES_USER -d $POSTGRES_DB -p 5432 <  db/3_seed_tokens.sql
```

### Application setup

#### 1. Dockerfile compose

To write a Dockerfile for running the application `analytics/app.py`

Please refer to `analytics/Dockerfile` for more information.

#### 2. Create YAMLs for application deployment


1. Create ConfigMap and Secret

    The ConfigMap to store the infromation of database accessing such as: DB_NAME, DB_USERNAME, DB_HOST, DB_PORT
    
    The Secret to store the sentative information such as DB_PASSWORD
    
    <sup><sub>Not recommend to store the password as plaintext here. However, in this practive we stored it as encode string from `base64`</sub></sup>

    Refer to: `deployment/configmap.yaml`

2. Create YAML for application deployment and service

    Compose the deployment yaml to describe the specification of Deployment such as container image (get from AWS ECR), liveness, readiness, listen port and the variable setting from ConfigMap and Secret above.

    Also compose the specification for the application service with port, protocol.

    Refer to: `deployment/coworking.yaml`

3. Apply these configuration to cluster

    ```bash
    kubectl apply -f deployment/configmap.yaml
    kubectl apply -f deployment/coworking.yaml
    ```

    You will see the services, pods when type command: `kubectl get all`



4. Test the connection

    By run command `kubectl get svc`, you will see the `EXTERNAL-IP` of `service/coworking`
    Use this `EXTERNAL-IP` for test the connection from host to application:

    ```bash
    curl <EXTERNAL-IP>:5153/health_check
    curl <EXTERNAL-IP>:5153/api/reports/daily_usage
    curl <EXTERNAL-IP>:5153/api/reports/user_visits
    ```

### CloudWatch setup

```bash
aws iam attach-role-policy \
--role-name eksctl-sa-cluster-nodegroup-my-nod-NodeInstanceRole-hHviYRUYCzrO \
--policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name sa-cluster
```

### Stand Out Suggestions
Please provide up to 3 sentences for each suggestion. Additional content in your submission from the standout suggestions do _not_ impact the length of your total submission.
1. Specify reasonable Memory and CPU allocation in the Kubernetes deployment configuration
2. In your README, specify what AWS instance type would be best used for the application? Why?
3. In your README, provide your thoughts on how we can save on costs?

