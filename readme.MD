
# 🏗️ Terraform Module: AWS Three-Tier Infrastructure for RoboShop

[](https://www.terraform.io/)
[](https://aws.amazon.com/)
[](https://opensource.org/licenses/MIT)

> A **reusable Terraform module** designed to provision a complete, scalable, and secure **three-tier infrastructure** on AWS, purpose-built for the **RoboShop** e-commerce application.

-----

## 📌 Purpose & Value Proposition

This module is the foundational layer for deploying the RoboShop application on AWS. It provisions the necessary cloud infrastructure across web, application, and database tiers, enabling:

  * **Modular Infrastructure Deployment:** Acts as a **drop-in dependency** for orchestrating complex RoboShop infrastructure pipelines, promoting clean separation of concerns.
  * **Accelerated Development:** Serves as a **production-ready reference module** for building or deploying similar 3-tier cloud architectures quickly and consistently.
  * **Full-Stack IaC Integration:** Seamlessly integrates into a comprehensive Infrastructure as Code (IaC) and configuration management workflow (Terraform + Ansible).

-----

## 🧱 Architecture Overview

This module provisions the core components of a classic three-tier web application architecture, ensuring scalability, security, and separation of concerns:

```mermaid
graph TD
    A[Client] --> B(Application Load Balancer<br>(Web Tier))
    B --> C{EC2 Instances<br>(Application Tier)}
    C --> D[RDS / DocumentDB / ElastiCache<br>(Database Tier)]

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#bfb,stroke:#333,stroke-width:2px
    style D fill:#fbb,stroke:#333,stroke-width:2px
```

  * **Web Tier:** Handled by the Application Load Balancer (ALB), routing traffic to the application servers.
  * **Application Tier:** Consists of EC2 instances running various RoboShop microservices (e.g., Catalogue, User, Cart, Payment).
  * **Database Tier:** Managed RDS instances providing persistent storage (MySQL, MongoDB, Redis).

-----

## 🔗 Dependency Usage

This module is specifically designed to be **consumed by** higher-level orchestration repositories. It provides the core infrastructure that other components build upon:

1.  **[roboshop-Infra-dev](https://github.com/Nallagachu/roboshop-Infra-dev)**:
      * This is the main infrastructure orchestrator that calls *this* module to provision AWS resources. It then proceeds to configure application services using Ansible.
2.  **[ansible-roboshop-roles-tf](https://github.com/Nallagachu/ansible-roboshop-roles-tf)**:
      * This module provisions a dedicated Ansible controller. This controller is then used to run playbooks on the EC2 instances provisioned by *this* `terraform-aws-roboshop` module.

-----

## 🚀 Prerequisites

Before you begin, ensure you have the following set up:

  * **AWS Account:** With sufficient permissions to create EC2, RDS, ALB, and Security Group resources.
  * **Terraform CLI:** Version `1.0.0` or higher installed locally.
  * **AWS CLI:** Configured with your AWS credentials.
  * **Existing VPC & Subnets:** This module deploys resources into an existing network; it does *not* create the VPC or subnets itself.
  * **EC2 Key Pair:** An SSH key pair uploaded to your target AWS region for secure access to EC2 instances.

-----

## 🛠️ Module Structure

```
terraform-aws-roboshop/
├── main.tf           # Defines core AWS resources (EC2 instances, RDS databases, ALBs, Security Groups).
├── variables.tf      # Declares all customizable input variables for the module.
├── outputs.tf        # Specifies output values (e.g., IPs, endpoints) for consumption by parent modules.
├── versions.tf       # Sets explicit Terraform and provider version constraints for compatibility.
└── README.md         # This comprehensive documentation file.
```

-----

## ⚙️ Key Features

This module provides the essential, pre-configured building blocks for your RoboShop application's backend:

  * **Network Integration:** Seamlessly deploys all resources within your existing VPC and specified subnet topology.
  * **Dedicated Application EC2s:** Provisions EC2 instances optimized for various app-tier components (e.g., Catalogue, User, Cart, Payment services).
  * **Managed Database Services:** Sets up robust AWS RDS instances for your database layer, supporting MySQL, PostgreSQL, and providing pathways for MongoDB (via DocumentDB/EC2) and Redis (via ElastiCache/EC2).
  * **Layered Security:** Configures granular Security Groups to control network traffic flow, enhancing the security posture of each tier.
  * **Traffic Distribution:** Integrates Application Load Balancers (ALBs) to efficiently distribute incoming traffic across your application instances.
  * **Actionable Outputs:** Generates crucial outputs like ALB DNS names, EC2 instance IPs, and database endpoints for easy integration with configuration management or other services.

-----

## 🔧 Input Variables

Customize your infrastructure deployment using these input variables. Required variables must be provided; optional variables have sensible defaults.

| Name                  | Description                                            | Type            | Required | Default             |
| :-------------------- | :----------------------------------------------------- | :-------------- | :------- | :------------------ |
| `env`                 | Deployment environment identifier (e.g., `dev`, `prod`). | `string`        | ✅       | `n/a`               |
| `vpc_id`              | ID of the existing VPC where resources will be deployed. | `string`        | ✅       | `n/a`               |
| `subnet_ids`          | List of existing subnet IDs for resource placement.    | `list(string)`  | ✅       | `n/a`               |
| `instance_type`       | EC2 instance type for application servers.             | `string`        | ❌       | `"t3.micro"`        |
| `db_instance_class`   | RDS instance type for the database.                    | `string`        | ❌       | `"db.t3.micro"`     |
| `db_engine`           | RDS engine type (`mysql`, `postgres`, `redis`, `mongodb`). Note: `redis`/`mongodb` imply EC2-based setups, not native RDS. | `string` | ✅ | `n/a` |
| `db_name`             | Name of the database to be created.                    | `string`        | ✅       | `n/a`               |
| `db_user`             | Admin username for the database.                       | `string`        | ✅       | `n/a`               |
| `db_password`         | Admin password for the database. **Sensitive.** | `string`        | ✅       | `n/a`               |
| `ami_id`              | Custom AMI ID for the EC2 instances. If not provided, it defaults to the latest Amazon Linux 2 AMI. | `string` | ❌ | `latest Amazon Linux 2` |
| `key_name`            | SSH Key Pair Name for EC2 instances. Required for SSH access. | `string` | ✅ | `n/a` |
| `app_port`            | Port on which application services will listen (e.g., 8080). | `number` | ❌ | `8080`              |
| `lb_port`             | Port for the Application Load Balancer listener.       | `number`        | ❌       | `80`                |

-----

## 📤 Output Variables

Upon successful deployment, these outputs provide essential information about the provisioned infrastructure, critical for subsequent configuration or integration:

| Name                    | Description                                                               |
| :---------------------- | :------------------------------------------------------------------------ |
| `app_instance_ids`      | List of IDs of the provisioned EC2 app instances.                         |
| `app_public_ips`        | List of Public IP addresses of the application EC2 instances.             |
| `db_endpoint`           | The connection endpoint (DNS name) of the RDS database.                   |
| `db_security_group_id`  | ID of the security group associated with the RDS instance.                |
| `alb_dns_name`          | The DNS name of the Application Load Balancer.                            |
| `alb_security_group_id` | ID of the ALB's security group.                                           |
| `app_security_group_id` | ID of the security group associated with the application EC2 instances.   |

-----

## 🚀 Example Usage

This module is typically invoked from a higher-level orchestrating Terraform project (e.g., `roboshop-Infra-dev`).

```hcl
# main.tf in your orchestrating project (e.g., roboshop-Infra-dev)

# Assuming you have a VPC module defined elsewhere and its outputs available
# module "vpc_network" {
#   source = "..."
# }

module "roboshop_aws_infrastructure" {
  source               = "github.com/Nallagachu/terraform-aws-roboshop" # Recommended: Use a specific Git ref or tag for production
  
  env                  = "dev"
  vpc_id               = module.vpc_network.vpc_id # Replace with your VPC module's output
  subnet_ids           = module.vpc_network.public_subnet_ids # Replace with your public subnets

  instance_type        = "t3.medium"
  db_instance_class    = "db.t3.medium"
  db_engine            = "mysql"
  db_name              = "roboshop"
  db_user              = "admin"
  db_password          = var.db_password # Best practice: Use a `var` and provide via TF_VARs or .tfvars file

  key_name             = "my-roboshop-ssh-key" # Ensure this key exists in your AWS account
  app_port             = 8080
  lb_port              = 80
}

# Example of consuming module outputs for subsequent steps (e.g., Ansible inventory)
output "application_server_ips" {
  description = "Public IPs of the RoboShop application EC2 instances."
  value       = module.roboshop_aws_infrastructure.app_public_ips
}

output "database_connection_endpoint" {
  description = "Connection endpoint for the RoboShop database."
  value       = module.roboshop_aws_infrastructure.db_endpoint
  sensitive   = true # Mark as sensitive if it contains credentials or should not be printed in logs
}

output "web_frontend_url" {
  description = "The DNS name of the Application Load Balancer to access the RoboShop application."
  value       = "http://${module.roboshop_aws_infrastructure.alb_dns_name}"
}
```

-----

## 🧩 End-to-End Workflow: IaC to Application

This module is a critical piece in a holistic DevOps workflow, enabling automated infrastructure provisioning and application configuration:

### 1\. **Provision Foundational Infrastructure (Terraform)**

  * **Action:** Execute `terraform init`, `terraform plan`, and `terraform apply` from your orchestrating project (e.g., `roboshop-Infra-dev`).
  * **Outcome:** This `terraform-aws-roboshop` module provisions all core AWS resources: EC2 instances for the application tier, RDS databases, Application Load Balancers, and necessary Security Groups, ready for application deployment.

### 2\. **Automate Application Configuration (Ansible)**

  * **Action:** The orchestrating project (or a dedicated CI/CD pipeline) leverages the outputs from this Terraform module to dynamically build an Ansible inventory.
  * **Integration:** The `ansible-roboshop-roles-tf` module can provision an Ansible controller if needed. Ansible playbooks (from your `roboshop-ansible` repository) are then executed on the newly provisioned EC2 instances, configuring the application components (e.g., installing software, setting up services, configuring database connections).

### 3\. **Deploy & Validate RoboShop Application**

  * **Action:** With infrastructure provisioned and applications configured, the RoboShop backend services become operational.
  * **Outcome:** The application is accessible via the ALB's DNS name, and all microservices communicate seamlessly within the established three-tier architecture.

-----

## 🤝 Contributing

We warmly welcome contributions to enhance this module\! Your input helps make it better for everyone.

You can contribute by:

  * **Extending Functionality:** Adding support for Autoscaling Groups, CloudWatch alarms, more advanced tagging strategies, or integrating additional AWS services.
  * **Improving Reliability:** Enhancing error handling, adding comprehensive unit/integration tests for Terraform, or refining resource definitions.
  * **Refactoring & Optimization:** Streamlining configurations, adopting new Terraform language features, or exploring multi-cloud portability.
  * **Documentation:** Improving existing descriptions, adding more detailed usage examples, or contributing guides.

Please open an [issue](https://www.google.com/search?q=https://github.com/Nallagachu/terraform-aws-roboshop/issues) first to discuss your proposed changes or submit a [pull request](https://www.google.com/search?q=https://github.com/Nallagachu/terraform-aws-roboshop/pulls) with clear explanations and justification.

-----

## 📄 License

This project is distributed under the **MIT License**. See the [LICENSE](https://www.google.com/search?q=LICENSE) file for the full text.

-----

## 👤 Maintainer

**Durgababu Nallagachu**

  * **GitHub:** [@Nallagachu](https://www.google.com/search?q=https://github.com/Nallagachu)
  * Feel free to connect for questions, collaboration opportunities, or suggestions for improvement\!

-----

## 🔗 Related Repositories

Explore the complete RoboShop infrastructure and automation ecosystem:

  * **🧱 Main Infrastructure Orchestration:** [roboshop-Infra-dev](https://github.com/Nallagachu/roboshop-Infra-dev)
  * **⚙️ Ansible Controller Module:** [ansible-roboshop-roles-tf](https://github.com/Nallagachu/ansible-roboshop-roles-tf)
  * **📦 Ansible Playbooks & Roles:** [roboshop-ansible](https://www.google.com/search?q=https://github.com/Nallagachu/roboshop-ansible) (Link to your actual Ansible roles repository)

-----