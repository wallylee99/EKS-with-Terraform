# Provision an EKS cluster using Terraform

This configuration will deploy an AWS's Elastic Kubernetes Service (EKS) cluster using Terraform. A VPC is created to provision the EKS cluster with the following architecture:

<img src="https://github.com/wallylee99/EKS-with-Terraform/blob/main/eks.png" height="100%">

To execute the Terraform workflow, run the following commands:
```
terraform init
terraform validate
terraform plan
terraform apply
```

To clean up and remove all the resources created from the workflow, run:
```
terraform destroy
```
