# AWSEKSPOC

**Travel POC**

This project makes use of Booking API and TravelPayouts API. Front end is developed using React and backend with Spring Boot. Resources are provisioned in AWS using Terraform.


![image](https://user-images.githubusercontent.com/23714753/149655570-a8f837be-0160-4f73-aba5-93c6052ee03e.png)
![image](https://user-images.githubusercontent.com/23714753/149655620-23db61a8-5a30-45ba-87c2-5c94c151a97f.png)
![image](https://user-images.githubusercontent.com/23714753/149655640-00a99ba1-dc84-4210-8e7b-21f555694256.png)
![image](https://user-images.githubusercontent.com/23714753/149655688-5bd5a7bf-9917-4352-ac5f-8b4bf8c01b95.png)
![image](https://user-images.githubusercontent.com/23714753/149656928-c511dc46-539f-4abe-9507-beac9d275ad8.png)



I have created microservices for API Gateway, Flight Service, Hotel Service, IATA Service.Authentication is done via Cognito. JWT is passed to API Gateway POD and once the token is validated, the respective service is called. API Gateways provides cross cutting concerns such as logging and security.  API Gateway service makes use of Spring Cloud Kubernetes,Spring Cloud Gateway . ALB can be integrated with Cognito but there are some limitations so went with API gateway pattern.

Terraform project TerraformEKSRDS creates the following resources

- VPC with 2 public subnets and 2 private subnets
- EKS Cluster
- IAM OIDC Identity Provider
- IAM roles for service account to access DynamoDB (set up access at POD level for AWS services)
- MySQL DB
- DynamoDB

Terraform project TerraformS3DNS creates the following resources

- S3 bucket for static website hosting
- IAM policy for S3

-  Cloud Front
-  SSL Certificate for the domain
-  Route 53 record for Cloud front
![img](https://documents.lucid.app/documents/b0103c7d-0e48-47b5-aca6-7df6e9d8e788/pages/0_0?a=967&x=51&y=-112&w=2397&h=2010&store=1&accept=image%2F*&auth=LCA%2083bdb1e46047c494f609799dd6745be1bc5e5ddf-ts%3D1642308984)

**Pre-Requisites:**

- Cognito User Pool (User Pool Id has to be added to jwt.aws.userPoolId in Api Gateway ConfigMap)
- Domain hosted at Route 53
**STEPS**

1. Register with https://rapidapi.com/tipsters/api/booking-com/ to get key for Booking API

2. Register with https://www.travelpayouts.com/developers/api to get key for Travel Payouts API

3. Substitute the keys in ConfigMaps in the respective service yaml file in the folder **KubernetesManifests**

4. Navigate to **TerraformEKSRDS** and execute the following command

   ```
   terraform init
   terraform plan
   terraform apply --auto-approve
   ```

5. Once the resources are created, get the MySQL endpoint and substitute in ConfigMap and external name service in 01-IataService.yaml 

6. Create DNS record for Ingress ALB in Route53

7. Navigate to **TerraformS3DNS** and execute the following command

   ```
   terraform init
   terraform plan
   terraform apply --auto-approve
   ```

   

 8. Navigate to **TravelServiceFrontend** and execute the following command

    ```
    npm run build
    ```

9. Final Step is applying Kubernetes manifests

   ```
   kubectl apply -f KubernetesManifests/.
   ```

10. Optionaly install kube-prometheus-stack helm chart

 Kube Prometheus Stack:

   ```
   helm repo add monitoring https://prometheus-community.github.io/helm-charts
   helm repo update
   helm pull monitoring/kube-prometheus-stack --untar=true
   
   ```

   Create customvalues.yaml with the following values

```yaml
grafana: 
  adminPassword: admin
  service:
    type: NodePort
```


   
