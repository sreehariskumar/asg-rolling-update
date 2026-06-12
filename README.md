# asg-rolling-update-using-ansible-dynamic-inventory


<img width="1536" height="1024" alt="1cb9e466-8ca5-4d7a-b175-5ad61e4070ae" src="https://github.com/user-attachments/assets/8330e7e4-972e-4752-9963-e7e56d70e3dc" />




### Feature:

If you’re looking to launch a website with high availability and scalability, Amazon EC2 Autoscaling Groups are a great option. However, managing multiple instances can be challenging, especially when you need to deploy new code or update the configuration.

We’ll explore how to use Ansible to automate the deployment process for a website running in an Amazon EC2 Autoscaling Group. We’ll cover how to set up a dynamic inventory, create multiple plays to manage different tasks, and handle error scenarios.


### Requirements:
- 1 x **ansible-master** server, with the following packages installed:
```s
yum install python3 puthon3-pip -y
pip3 install ansible boto botocore boto3
ansible-galaxy collection install amazon.aws
```
(**boto**, **botocore**, and **boto3** are AWS SDKs (software development kits) for Python that Ansible uses to interact with AWS services.

**Ansible Galaxy**, which is a repository for sharing and downloading Ansible collections, to install the amazon.aws collection. The amazon.aws collection provides a set of modules and plugins that enable Ansible to manage AWS resources.)

- A launch configuration to create an autoscaling group to spin up instances
- An Elastic LoadBalancer


![6](https://user-images.githubusercontent.com/68052722/221768517-c2c8065b-3e7a-4dc2-a426-d7f444a65c74.png)
All instances launched from an autoscaling group will have a default tag with **Key = aws:autoscaling:groupName** & **Value** with the name of the autoscaling group.

### Dynamic Inventory
Dynamic inventory is a feature in Ansible that allows for the automatic generation of inventory based on some external system, such as a cloud provider, virtualization platform, or configuration management database.


### Playbook
This playbook has two plays:

### First Play
The first play is named “fetching instance details”. This play fetches the EC2 instance details of the instances in the Auto Scaling Group (ASG) and registers them to the Ansible inventory.

#### Tasks:
- **gathering instance details**: This task uses the **“amazon.aws.ec2_instance_info”** module to fetch the EC2 instance details of instances in the ASG, and filters the instances based on their state and tags. The instances that match the filters are registered to the Ansible inventory using the **“register”** keyword.
- **creating dynamic inventory**: This task uses the **“add_host”** module to dynamically add the instances that were registered in the previous task to the **“asg_instances”** group in the Ansible inventory. This task sets some necessary SSH connection parameters and specifies the private key file to connect to the instances.
- The **serial** keyword is used to specify the number of hosts to run the playbook on at the same time. In this case, serial: 1 is used to run the playbook on each host sequentially, meaning that the tasks in the playbook will be executed one at a time on each host instead of running them all simultaneously.
We’ve used the same template files from a previous article.

### Second Play
The second play is named **“Deploying a site from github repo”**. This play installs some necessary packages, clones a GitHub repository, configures Apache web server, and copies the website files to the document root of the web server. This play is executed on all instances in the “asg_instances” group.

#### Tasks:
- **installing packages**: This task installs the necessary packages required for the Apache web server, PHP, and Git.
- **creating conf from template**: This task uses the “template” module to create an Apache web server configuration file based on a Jinja2 template.
- **creating virtualhost from template**: This task uses the “template” module to create a virtual host configuration file based on a Jinja2 template.
- **creating document root**: This task creates the document root directory for the website files.
- **creating cloning directory**: This task creates a directory for cloning the GitHub repository.
- **cloning from repo**: This task uses the “git” module to clone the specified GitHub repository.
- **stopping instances**: This task stops the Apache web server on all instances if the repository was cloned, indicating that new code has been deployed.
- **connection drain waiting**: This task waits for the specified amount of time to let the existing connections drain before restarting the web server.
- **copying contents to document root**: This task copies the website files from the cloned repository to the document root of the web server.

### Handlers:
- **apache-restart**: This handler restarts the Apache web server service after the website files have been copied to the document root.
- **apache-reload**: This handler reloads the Apache web server configuration after the website configuration files have been created or updated.
- **up-delay**: This handler waits for the specified amount of time before restarting the web server service to allow the new instances to come up and start serving traffic.


### Result
You could see the contents of the website by accessing the end-point of the load balancer you’ve created. You can also point the end-point to a DNS name and access the domain name instead.

### Conclusion
Overall, this article has demonstrated how Ansible and AWS can be used together to automate the deployment process, reducing the time and effort required to deploy and manage web applications. By using Ansible to automate the deployment process, developers can focus on writing code and building new features, while Ansible handles the deployment and configuration of the application on the infrastructure.
