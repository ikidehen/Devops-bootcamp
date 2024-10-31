# Prerequisites for Deploying the Prometheus Stack using Docker Compose

# Documentation

- Spin up an Ubuntu instance and connect the instance via ssh

- Clone the project repository to instance

```
git clone https://github.com/TobiOlajumoke/prometheus-observability-stack
```

- The project code is present in the prometheus-observability-stack folder. cd into the folder.

![11](img10/11.png)

This  is the project structure and config files:

```

├── Makefile
├── README.md
├── alertmanager
│   └── alertmanager.yml
├── docker-compose.yml
├── prometheus
│   ├── alertrules.yml
│   ├── prometheus.yml
│   └── targets.json
└── terraform-aws
    ├── README.md
    ├── modules
    │   ├── ec2
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   ├── user-data.sh
    │   │   └── variables.tf
    │   └── security-group
    │       ├── main.tf
    │       ├── outputs.tf
    │       └── variables.tf
    ├── prometheus-stack
    │   ├── main.tf
    │   ├── outputs.tf
    │   └── variables.tf
    └── vars
        └── ec2.tfvars
        
```

Spin up an ec2 instance and create and attach a role to the instace

![1](img10/1.png)

![2](img10/2.png)

![3](img10/3.png)

![4](img10/4.png)

![5](img10/5.png)

![6](img10/6.png)

![7](img10/7.png)

![8](img10/8.png)

- After creating the instance attath it to the role created earlier

![9](img10/9.png)

![10](img10/10.png)

### Connect to your instance via ssh

- ssh into the instance

- install terraform and run the commands 

```
sudo snap install terraform --classic
```

```
cd terraform-aws/vars
```

![12](img10/12.png)

- Run the commands

To Modify the values of ec2.tfvars file present in the terraform-aws/vars folder. You need to replace the values highlighted in bold with values relevant to your AWS account & region.

```
vi ec2.tfvars
```

```
# EC2 Instance Variables
region         = "us-west-2"
ami_id         = "ami-0aff18ec83b712f05"
instance_type  = "t2.large"
key_name       = "add keyname"
instance_count = 1
volume-size = 20

# VPC id
vpc_id  = "add a vpc"
subnet_ids     = "add a subnet"

# Ec2 Tags
name        = "prometheus-stack"
owner       = "devops-mastery"
environment = "dev"
cost_center = "devops-project"
application = "monitoring"

# CIDR Ingress Variables
create_ingress_cidr        = true
ingress_cidr_from_port     = [22, 80, 443, 9090, 9100, 9093, 3000]  # List of from ports
ingress_cidr_to_port       = [22, 80, 443, 9090, 9100, 9093, 3000]  # List of to ports
ingress_cidr_protocol      = ["tcp", "tcp", "tcp", "tcp", "tcp", "tcp", "tcp"]        # Protocol for all rules (you can add more if needed)
ingress_cidr_block         = ["0.0.0.0/0", "0.0.0.0/0", "0.0.0.0/0", "0.0.0.0/0", "0.0.0.0/0", "0.0.0.0/0", "0.0.0.0/0"]
ingress_cidr_description   = ["SSH", "HTTP", "HTTPS", "Prometheus", "Node-exporter", "Alert manager", "Grafana"]

# CIDR Egress Variables
create_egress_cidr    = true
egress_cidr_from_port = [0]
egress_cidr_to_port   = [0]
egress_cidr_protocol  = ["-1"]
egress_cidr_block     = ["0.0.0.0/0"]
```

- fill it with the correct variable 

![13](img10/13.png)

Now we can provision the AWS EC2 & Security group using Terraform.

```
cd ../prometheus-stack
```

![14](img10/14.png)

```
terraform fmt
```

![15](img10/15.png)

```
terraform init
```

![16](img10/16.png)

```
terraform validate
```

![17](img10/17.png)

- Execute the plan and apply the changes.

```
terraform plan --var-file=../vars/ec2.tfvars
```

![18](img10/18.png)

```
terraform apply --var-file=../vars/ec2.tfvars
```

Before typing ‘yes‘ make sure the desired resources are being created. After running Terraform Output should look like the following:

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

![19](img10/19.png)

## CONNECT TO YOUR NEW INSTANCE

![20](img10/20.png)

![21](img10/21.png)

- let's run the following command on our instance

We will check the cloud-init logs to see if the user data script has run successfully.

```
tail /var/log/cloud-init-output.log
```

![22](img10/22.png)

- Next is to verify docker and docker compose versions

```
sudo docker version
```

```
sudo docker-compose version
```

![23](img10/23.png)

![24](img10/24.png)

### Deploy Prometheus Stack Using Docker Compose

- clone the project code repository to the server

```
git clone https://github.com/TobiOlajumoke/prometheus-observability-stack
```

![25](img10/25.png)

```
cd prometheus-observability-stack
```

![26](img10/26.png)

Execute the following make command to update server IP in prometheus config file. Because we are running the node exporter on the same server to fetch the server metrics. We also update the alert manager endpoint to the servers public IP address.

```
make all
```

![27](img10/27.png)

- Bring up the stack using Docker Compose. It will deploy Prometheus, Alert manager, Node exporter and Grafana

```
sudo docker-compose up -d
```

If successful you will see output saying running 5/5

![28](img10/28.png)

Now, with your servers IP address you can access all the apps on different ports.

- Prometheus: http://your-public-ip-address:9090

- Alert Manager: http://your-public-ip-address:9093

- Grafana: http://your-public-ip-address:3000

Now the stack deployment is done. The rest of the configuration and testing will be done the using the GUI.

### Validate Prometheus Node Exporter Metrics

If you visit http://your-public-ip-address:9090, you will be able to access the Prometheus dashboard as shown below.

Validate the targets, rules and configurations as shown below. The target would be Node exporter url.

![29](img10/29.png)

- Click on configuration

![30](img10/30.png)

- Click on rules

![31](img10/31.png)

- Clickon targets

![32](img10/32.png)

![33](img10/33.png)

- Click on graph at the top bar

![34](img10/34.png)

validating prometheus rules and targets

Now lets execute a promQL statement to view node_cpu_seconds_total metrics scrapped from the node exporter.

```
avg by (instance,mode) (irate(node_cpu_seconds_total{mode!='idle'}[1m]))
```

![35](img10/35.png)

![36](img10/36.png)

executing a promQL statement to get graph

### Now lets configure Grafana dashboards for the Node Exporter metrics.

Grafana can be accessed at: http://your-ip-address:3000

Use admin as username and password to login to Grafana. You can update the password in the next window if required.

Now we need to add prometheus URL as the data source from Connections→ Add new connection→ Prometheus → Add new data source.

![37](img10/37.png)

![38](img10/38.png)

![39](img10/39.png)

![40](img10/40.png)

![41](img10/41.png)

![42](img10/42.png)

![43](img10/43.png)

![44](img10/44.png)

![45](img10/45.png)

![46](img10/46.png)

### Simulate & Test Alert Manager Alerts

You can access the Alertmanager dashbaord on http://your-ip-address:9093

![47](img10/47.png)

- Go back to prometheus, click on Alert on the top bar

![48](img10/48.png)

As you can see, all the alerts are in inactive stage. To test the alerts, we need to simulate these alerts using few linux utilities.

You can also check the alert rules using the native promtool prometheus CLI. We need to run promtool command from inside the prometheus container as shown below. run the commands below in the prometheus server

```
sudo docker exec -it prometheus promtool check rules /etc/prometheus/alertrules.yml
```

![50](img10/50.png)

```
dd if=/dev/zero of=testfile_16GB bs=1M count=16384; openssl speed -multi $(nproc --all) &
```

![51](img10/51.png)

Now we can check the Alert manager UI to confirm the fired alerts.

![52](img10/52.png)

Now let’s rollback the changes and see the fired alerts has been resolved.

```
rm testfile_16GB && kill $(pgrep openssl)
```

![53](img10/53.png)

### Cleanup The Setup

To tear down the setup, execute the following terraform command from your workstation.

```
cd prometheus-observability-stack/terraform-aws/prometheus-stack
```

![54](img10/54.png)

```

terraform destroy --var-file=../vars/ec2.tfvars
```

![55](img10/55.png)

![56](img10/56.png)

# The End 
