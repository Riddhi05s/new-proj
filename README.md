registration-app
<br>
Test33


------------------------------------------------------------------------------------------------------------------------

### Eks-githubactions
- Create Developer-server and create local git repository
- Create Github Repository
- Create Access-Secret key for AWS policies

![image](https://github.com/user-attachments/assets/840a25cc-5620-4db2-8fee-8bfc49f429ea)

![image](https://github.com/user-attachments/assets/2bc94716-b688-43b9-ac1b-d91428e1f9c6)

<img width="634" alt="image" src="https://github.com/user-attachments/assets/2bf9451c-f7c8-4324-b77f-3f071719911f">

![image](https://github.com/user-attachments/assets/8b0d60aa-0a54-4617-aa17-1764e3d28651)
![image](https://github.com/user-attachments/assets/5032f7c5-5e8d-496c-9a7d-d37b583da7a6)
![image](https://github.com/user-attachments/assets/d85fb5e1-21e2-4238-8ba4-8baf4cffdf9a)
![image](https://github.com/user-attachments/assets/7af9ea77-c75c-4426-a9dd-e691c1a3390a)
![image](https://github.com/user-attachments/assets/97cdb408-94e5-4193-bca0-e9cee79b20c8)

 
- Create eks-server and install kubectl and eksctl

```bash
ssh-keygen

aws-configure

eksctl create cluster \
  --name my-cluster \
  --region ap-south-1 \
  --nodegroup-name my-nodes \
  --node-type t2.small \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --managed

eksctl create cluster --name adinath-cluster-914 --region ap-south-1 --version 1.29 --vpc-public-subnets subnet-044c44fef8814b390,subnet-065909e8f402e86a7 --node-type t2.micro --nodes-min 2 --ssh-access --ssh-public-key /root/.ssh/id_rsa.pub
```
- in github repository, manifests directory add hello-app-deployment.yaml and hello-app-service.yaml

  **hello-app-deployment.yaml**
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: regapp-deployment
    labels:
       app: regapp
  
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: regapp
  
    template:
      metadata:
        labels:
          app: regapp
      spec:
        containers:
        - name: regapp
          image: 241533119861.dkr.ecr.ap-south-1.amazonaws.com/eksdeployment:latest
          imagePullPolicy: Always
          ports:
          - containerPort: 8080
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
  ```

  **hello-app-service.yaml**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: regapp-service
    labels:
      app: regapp
  spec:
    selector:
      app: regapp
  
    ports:
      - port: 8080
        targetPort: 8080
  
    type: LoadBalancer
  ```
- in local git repo add file deploy.yml in .github/workflows
```yaml
name: Deploy to ECR

on: 
  push:
    branches: [ main ]

env:
  ECR_REPOSITORY: eksdeployment
  EKS_CLUSTER_NAME: adinath-cluster-914
  AWS_REGION: ap-south-1

jobs:
  build:
    name: Deployment
    runs-on: ubuntu-latest

    steps:

    - name: Set short commit SHA
      id: commit
      uses: prompt/actions-commit-hash@v2

    - name: Check out code
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with: 
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Set up JDK 14
      uses: actions/setup-java@v1
      with:
        java-version: 14

    - name: Build project with Maven
      run: mvn -B package --file pom.xml

    - name: Login to amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag and Push
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ${{ env.ECR_REPOSITORY }}
        IMAGE_TAG: "latest"
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        echo "Pushing image to ECR..."
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"

    - name: Update kube config
      run: aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION

    - name: Deploy to EKS
      env: 
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ steps.commit.outputs.short }}
      run: |
        kubectl rollout restart deployment/regapp-deployment
        kubectl apply -f manifests/hello-app-deployment.yaml
        kubectl apply -f manifests/hello-app-service.yaml
```
- In EKS server *kubectl get svc* you'll get external IP for service. View that IP with 8080 port and /webapp extension
- try to make changes in jsp file of source code, and check if those changes are automatically applied in final output.

![image](https://github.com/user-attachments/assets/3eff75c9-d698-414f-8cb8-316845d7e39e)

<img width="959" alt="image" src="https://github.com/user-attachments/assets/19bc09c9-9250-4963-8a5c-3aea1c78375c">
<img width="959" alt="image" src="https://github.com/user-attachments/assets/30b6a351-7555-4401-80ae-d2f585500fb4">
