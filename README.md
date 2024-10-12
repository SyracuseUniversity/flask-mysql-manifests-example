# flask-mysql-manifests-example
Kubernetes manifests for deploying flask-mysql-example repository. 

> [!NOTE]
> If you would like to use these manifests for ArgoCD, be sure to fork this repository into your own repository.

## Secrets
> [!IMPORTANT]
> You must setup secrets before deploying application manifests.
### MariaDB Root Password
This will store the root password for MariaDB.

Replace `<NAMESPACE>` with the namespace you'll be deploying your manifests to, and replace `<CHANGE_ME>` with a 24 character alphanumeric string:

```bash
kubectl create secret -n <NAMESPACE> generic mariadb-root-password --from-literal=password=<CHANGE_ME>
```

### Flask App Secret
This will store the app secret for flask. 

Replace `<NAMESPACE>` with the namespace you'll be deploying your manifests to, and replace `<CHANGE_ME>` with a 24 character alphanumeric string:

```bash
kubectl create secret -n <NAMESPACE> generic flask-app-secret --from-literal=secret=<CHANGE_ME>
```

### regcred
You should not have to set this up if your project namespace was set up by iSchool Technology Services. 

If you receive errors that your application is unable to pull image, then you will need to reach out to make sure this was configured properly.

### tls-certs
You should not have to set this up if your project namespace was set up by iSchool Technology Services. 

If you receive errors that your application is unable to initialize, then you will need to reach out to make sure this was configured properly.

## Deploying Manifests
### Pre-deployment
Before deploying manifests, you should change a few values.

#### [mariadb-deployment.yaml](./mariadb-deployment.yaml)
Before deploying this application, you should request a "Persistent Volume" be setup in your project's namespace. The adminsitrators will provide you with a Volume Claim Name. You will replace `<VOLUME_CLAIM>` with the Volume Claim Name you were provided:

```yaml
...
      volumes:
        - name: mariadb-storage
          persistentVolumeClaim:
            claimName: <VOLUME_CLAIM>
      containers:
...
```

#### [app-deployment.yaml](./app-deployment.yaml)
If you plan on deploying a different application than the one in this example, you will need configure the flask-app container to pull container image of your choosing. you can specify the container image you'd like to use by updating the `image` attribute.

> [!NOTE]
> If you plan to deploy an image not stored in harbor.ischool.syr.edu, you will need to setup the regcred secret to contain your container registry's authentication credentials.

```yaml
...
      containers:
      - name: flask-app
        image: harbor.ischool.syr.edu/examples/flask-mysql-example:latest
        imagePullPolicy: Always
...
```

#### [mariadb-nodeport.yaml](./mariadb-nodeport.yaml)
If the nodeport doesn't deploy correctly, change the `nodePort` to a different value from 31000-32000:
```yaml
...
  - port: 3306
    nodePort: 31081
    protocol: TCP
...
```

#### [app-nodeport.yaml](./app-nodeport.yaml)
If the nodeport doesn't deploy correctly, change the `nodePort` to a different value from 31000-32000:
```yaml
...
  - port: 8443
    nodePort: 31080
    protocol: TCP
...
```

### Deploy using ArgoCD
> [!IMPORTANT]
> Before deploying, you will need to have a project configured in ArgoCD for your application. Please reach out to iSchool Technology Services to have your project provisioned.

1. Login to ArgoCD using your SU NetID: https://argocd.ischool.syr.edu
2. Click on the *"+ NEW APP"* button:
  - General
    - **Application Name**: set this to a unique name, using lowercase letters, numbers, or dashes ("-").
    - **Project**: Select the project name that was provisioned for your application.
    - **Sync Policy**: Select *Automatic*.
  - Source
    - **Repository URL**: Select the url for your projects manifests
    - **Path**: /
  - Destination
    - **Cluster URL**: https://kubernetes.default.svc
    - **Namespace**: this will be the namespace that was create for your project.
3. Click *CREATE*

You will need to refresh the page to see your application tile. Click on the application tile and you will see all the resources being deployed. If your application does reach a healthy state within 5-10 minutes, reach out to the iSchool Technology Services team to further debug your application.

## Setting up Database
You will need to login to mysql a create the database before the example application will work. To login, you will need to install a MySQL client. 

Once you have a MySQL client installed, ask iSchool Technology Services for the hostname of the node to connect to your db instance.

Once you have the hostname you can connect to the mariadb instance with the following connection string. This will ask for the root password you set earlier in the secrets section:

```bash
mysql --host <node name> --port 31081 -u root -p
```

once logged in, use the following line to create the *logbook* database:
```sql
CREATE DATABASE logbook;
```

after that you can exit by typing `exit` and pressing enter.

## Setting up a domain name to point to your application
to setup a domain name to point to your application, reach out to the iSchool Technology Services team.
