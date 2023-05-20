# Coworking Space Service Deployment Process

Coworking Space service is python based application that uses postgres as its database. App and database is hosted on AWS EKS while AWS cloudwatch is used as its logging service. <br>
Please understand the deployment pipeline first and then read the release instructions: 

### Deployment pipeline steps: 
1. On AWS EKS, a cluster named `coworking` is made with proper IAM roles and permissions. 
2. Using helm, postgres database is setup on the cluster. Helm automatically stores postgres password in cluster secrets named as `my-postgresql`. We use this password directly from there instead of hard coding it anywhere in the repository.
3. For migrating the database queries, Easiest way is to first copy the queries to postgres pod and execute the queries file from there only. This step is handled by devops-team. 
4. Once postgres-db is ready, we have setup a AWS Codebuild pipeline. This pipeline is triggerred once a PR is merged into main branch and the image is pushed to AWS ECR repository. 
5. Coworking-space repository has k8s-scripts in its deployment directory, that picks up the latest image from ECR and deploys it on `coworking` cluster.
6. The application logs are available in AWS Cloudwatch for any debugging purposes. Std-out logs are directed towards AWS cloudwatch using cloudwatch-agent. This agent is deployed on a separate kubernetes namespace: `amazon-cloudwatch`

### Deployment Guidelines:
1. Raise a PR from feature branch to main. Once it is merged, AWS Codebuild will be triggered. 
2. This pipeline builds your code and fails if there is any error. Built image is then pushed to AWS ECR repository.
3. After this, please reach out to us for production-deployment. We will update the appropriate buildNumber of your image in [analytics-deployment.yaml](deployment/analytics-deployment.yaml) and proceed with production-release using `kubectl apply` commands
4. Once production-release is done, you can verify the logs on AWS cloudwatch.
5. Environment variables are to be stored in [configMap.yaml](deployment/configmap.yaml) file and if there are any changes, please reach out to devops team. 
6. Secrets are managed in `coworking` cluster directly by devops team, if there are any changes, please reach out. Don't push any secret in plain text to repository directly.
7. For any pod level or db level debugging, reach out to us. We can directly login into the pod using `kubectl exec -t <pod-name> -- bash` command. For security reasons, we can't grant direct access to people other than devops team.