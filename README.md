# Migration of Nginx Webserver to Highly Available Docker Containers on AWS using Docker Swarm

![picture with the words Cloud Project and icons for the services used](https://miro.medium.com/v2/resize:fit:720/format:webp/0*W5iPkqkzUbmCPw4G.jpg)

In this Cloud Migration project based on a real-world scenario, I acted as a DevOps Engineer migrating to AWS,
a restaurant application that ran on-premises servers using Nginx.

However, I needed to keep the Nginx solution and deploy it through containers, with high availability, and using Docker 
Swarm as an orchestration solution. Besides, I used the Elastic Load Balancer (ELB) to ensure high availability on the three 
EC2 instances, located in different availability zones.

![picture with a graph showing the solution architecture](https://miro.medium.com/v2/resize:fit:720/format:webp/0*S6cw5RHp_a2YMrJD.jpg)

The first step was creating the EC2 instances on AWS using three different Availability Zones, to guarantee high availability. After that, I’ve created a 
repository on Amazon Elastic Container Registry, which will store our docker image later. While the instances were turning on and running the setup commands,
I’ve created a Target Group with the three machines and an Elastic Load Balancer, that will distribute the traffic later. In this process, I also updated
the Security Group, opening the necessary access ports for this project.

On the first instance I downloaded the project files. Using terminal, I created a Dockerfile using nginx as a base image and copying the app folder. 
Using the commando docker build, I’ve created our app image. For educational purposes, I’ve created a programmatic access user and inserted the keys into 
the instance’s AWS CLI, so it could access my repository on ECR. Normally this process it’s not necessary, since the best practice here is using a role on the
instances — this way they have direct and secure access to the needed credentials.

With the imaged pushed to the repository and already stored there, I’ve used the commando docker swarm init to establish the EC2 instance I was using as the
Manager Node, and using the token generated there to setup the other two machines as Worker Nodes. Instantly, the three machines were already connected to
each other through Docker’s automatically created network. With the ground ready, I just needed to use docker service create, using a -p flag to open the ports, 
a name flag and a -with-registry-auth followed by my ECR-stored image URI. With that flag, the Manager Node is responsible to pass on the credentials needed to
the Worker Nodes. With the commands done, the app was already live.

Initially the app was only available on one of the VMs. Using docker service scale, Docker generated copies of the container on the other VMs, quickly escalating 
our app, which became available on the other two machines.

To test recovery services, I ran some tests. First, I’ve forcibly removed the container running on VM 02, which resulted in Docker automatically creating a new
container in less than 30 seconds. On the second test, I’ve changed the Manager Node availability, so it could not receive containers or services anymore. 
Docker quickly created a container on VM 02, respecting the rule of 3 replicas I’ve setup previously using docker service scale.

On the last test, which was more radical, I’ve shutdown VM 02. Docker quickly detected and automatically created two more containers on VM 03, keeping the necessary 
number of replicas. After turning on VM 02 again, Docker quickly detected it and reconnected it as a Worker Node, but didn’t rebalanced the containers. To change that,
I used docker service update with a force flag — Docker reboot the entire Swarm but balanced the containers respecting the needed number of replicas and machine’s availability.

Docker is, without doubt, one of the most amazing apps I’ve known. The ability to run contained apps in seconds, using few resources and capable of escalating to big numbers,
make Docker one of the most powerful and useful tools in the arsenal of any dev or DevOps professional today.

