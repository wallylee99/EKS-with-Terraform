<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__html"><h1 id="provisioning-an-eks-cluster-using-terraform">Provisioning an EKS cluster using Terraform</h1>
<p>In this exercise, I will deploy an AWS’s Elastic Kubernetes Service (EKS) cluster using Terraform and configure kubectl to verify the cluster is ready to use. Kubernetes is an open-source container orchestration system used to manage and deploy containerized applications. A Kubernetes cluster is a collection of nodes that run containerized applications. With AWS EKS, I can run my Kubernetes applications on AWS using their Elastic Compute Cloud (EC2) and take advantage of its scalability and availability using services such as application load balancers (ALB) and auto scaling group (ASG).</p>
<h2 id="preparing-terraform-configuration-files">Preparing Terraform Configuration Files</h2>
<p>This configuration will creates a VPC to provision an EKS cluster with the following architecture:</p>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/eks-2.png" alt=""></p>
<p>A vpc is created with three public subnets and three private subnets across two availability zones:</p>
<pre><code>module "vpc" {
    source = "terraform-aws-modules/vpc/aws"
    version = "3.14.2"

    name = "terraform-eks-vpc"

    cidr = "10.0.0.0/16"
    azs = slice(data.aws_availability_zones.available.names, 0, 3)

    private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
    public_subnets = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]
</code></pre>
<p>The EKS cluster will contain three nodes distributed across two node groups, with each compute node being of t2.micro type:</p>
<pre><code>    eks_managed_node_groups = {
        one = {
            name = "node-group-1"
            instance_types = ["t2.micro"]
            min_size = 1
            max_size = 3
            desired_size = 2
        }

        two = {
            name = "node-group-2"
            instance_types = ["t2.micro"]
            min_size = 1
            max_size = 3
            desired_size = 2
        }
    }
</code></pre>
<h2 id="running-the-terraform-code">Running the Terraform Code</h2>
<p>A terraform workflow consists of four main steps:</p>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/workflow.png" alt=""></p>
<p><strong>Init</strong>  – Prepare the workspace and download the required modules and providers so Terraform can apply the configuration I have defined in various configuration files, run this command:</p>
<pre class=" language-hcl"><code class="prism  language-hcl">$terraform init
</code></pre>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/init-1.png" alt=""></p>
<p><strong>Validate</strong>  – Terraform will verify the syntax of your terraform configuration files</p>
<pre class=" language-hcl"><code class="prism  language-hcl">$terraform validate
</code></pre>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/validate-1.png" alt=""></p>
<p><strong>Plan</strong>  – It allows you to preview the changes Terraform will make before apply them</p>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/plan-1024x414.png" alt=""></p>
<p><strong>Apply</strong>  – Terraform will deploy the changes by creating or updating your resources in AWS</p>
<pre class=" language-hcl"><code class="prism  language-hcl">$terraform apply
</code></pre>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/apply-2.png" alt=""></p>
<p>It takes about 10-15 minutes to create the cluster. When it is completed, I can see my EKS cluster in AWS console.</p>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/aws_eks.png" alt=""></p>
<h2 id="configure-kubectl">Configure KubeCTL</h2>
<p>Kubernetes provides a command line tool called  <strong>kubectl</strong>  for communicating with a Kubernetes cluster’s control plane, using the Kubernetes API. To use it first we need to a configuration file kubeconfig:</p>
<pre class=" language-hcl"><code class="prism  language-hcl">$aws eks update-kubeconfig --region region_name --name cluster_name
</code></pre>
<p>To validate and test my configuration to the master node:</p>
<pre class=" language-hcl"><code class="prism  language-hcl">$kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   172.20.0.1   &lt;none&gt;        443/TCP   5h39m

</code></pre>
<p>To view the status of my nodes:</p>
<pre class=" language-hcl"><code class="prism  language-hcl">
    $kubectl get nodes custom-columns=Name:.metadata.name,nCPU:.status.capacity.cpu,Memory:.status.capacity.memory
    Name                                            nCPU   Memory
    ip-10-0-1-150.ca-central-1.compute.internal     1      2031268Ki
    ip-10-0-1-189.ca-central-2.compute.internal     1      2031268Ki
    ip-10-0-2-241.ca-central-2.compute.internal     1      2031268Ki
    ip-10-0-2-172.ca-central-2.compute.internal     1      2031268Ki

</code></pre>
<h2 id="destroy-the-eks-cluster">Destroy the EKS Cluster</h2>
<p>To destroy the whole infrastructure that Terraform has created, simply use the following command:</p>
<pre class=" language-hcl"><code class="prism  language-hcl">$terraform destroy
</code></pre>
<p><img src="https://www.wallacel.com/wp-content/uploads/2024/05/destroy-1.png" alt=""></p>
</div>
</body>

</html>
