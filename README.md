# Terraform Docker IaC



## Infrastructure as Code with Terraform and Docker

This project demonstrates Infrastructure as Code (IaC) by provisioning and managing a local Docker-based Nginx web server using Terraform.

The project uses the `kreuzwerker/docker` Terraform provider to communicate with the local Docker Engine. Terraform manages the Docker image and container lifecycle, including creation, updates, state tracking, and destruction.



## Objective

Provision a local Docker container using Terraform and demonstrate the complete infrastructure lifecycle:

```text
Terraform configuration → Plan → Apply → Verify → State → Destroy
```

The project follows the Task 3 requirements: use the Docker provider, create a container, run `terraform init` and `apply`, use `terraform plan`, inspect Terraform state, and destroy the infrastructure using `terraform destroy`.



## Technologies Used

|Technology|Version / Details|
|-|-|
|Terraform|1.16.5|
|Docker Desktop|4.93.0|
|Docker Engine|29.8.1|
|Docker Provider|`kreuzwerker/docker` 4.6.0|
|Container|Nginx Alpine|
|Host OS|Windows|

## 

## Architecture

```text
                    Terraform
                        |
                        v
              Docker Provider
                        |
          +-------------+-------------+
          |                           |
          v                           v
    nginx:alpine              Docker Container
                              terraform-nginx
                                    |
                                    v
                              Port Mapping
                              8081 → 80
                                    |
                                    v
                              Nginx Web UI
```

## Project Structure

```text
terraform-docker-iac/
│
├── main.tf
├── .gitignore
├── .terraform.lock.hcl
├── README.md
│
├── site/
│   └── index.html
│
├── execution-logs/
│   ├── terraform-plan.txt
│   ├── terraform-state.txt
│   ├── terraform-destroy.txt
│   ├── docker-ps.txt
│   ├── terraform-state-after-destroy.txt
│   └── docker-ps-after-destroy.txt
│
└── screenshots/
    └── project execution screenshots
```

> The `.terraform/` directory and Terraform state files are excluded from Git using `.gitignore`. The `.terraform.lock.hcl` provider lock file is retained.



## Terraform Configuration

The project defines two Docker resources.



### Docker Image

```hcl
resource "docker\\\_image" "nginx" {
  name         = "nginx:alpine"
  keep\\\_locally = true
}
```

This manages the Nginx Alpine image required by the container.



### Docker Container

```hcl
resource "docker\\\_container" "nginx" {
  name  = "terraform-nginx"
  image = docker\\\_image.nginx.image\\\_id

  ports {
    internal = 80
    external = 8081
  }

  volumes {
    host\\\_path      = abspath("${path.module}/site")
    container\\\_path = "/usr/share/nginx/html"
    read\\\_only      = true
  }
}
```

The local `site/` directory is mounted into Nginx as read-only content, allowing the container to serve the custom Infrastructure as Code dashboard.



## Infrastructure Lifecycle



### 1\. Initialize Terraform

```powershell
terraform init
```

Initializes the Terraform working directory and installs the required Docker provider.

### 2\. Format and Validate

```powershell
terraform fmt
terraform validate
```

The configuration was formatted and successfully validated before provisioning.

### 3\. Preview Changes

```powershell
terraform plan
```

`terraform plan` previews the changes Terraform intends to make without modifying the infrastructure.

### 4\. Provision Infrastructure

```powershell
terraform apply
```

The configuration successfully provisioned:

* Docker image: `nginx:alpine`
* Docker container: `terraform-nginx`
* Port mapping: `localhost:8081 → container:80`

### 5\. Verify the Container

```powershell
docker ps
```

The `terraform-nginx` container was verified as running and the application was accessible at:

```text
http://localhost:8081
```

### 6\. Inspect Terraform State

```powershell
terraform state list
```

The state contained:

```text
docker\\\_container.nginx
docker\\\_image.nginx
```

Terraform state allows Terraform to track the infrastructure resources managed by the configuration.

### 7\. Destroy Infrastructure

```powershell
terraform destroy
```

The infrastructure was successfully destroyed:

```text
Destroy complete! Resources: 2 destroyed.
```

After destruction, the Terraform state was empty and the `terraform-nginx` container was no longer running.



## Custom Infrastructure Dashboard

A custom Nginx dashboard was created under `site/index.html` to provide a professional visual representation of the provisioned infrastructure.

The dashboard displays:

* Terraform status
* Docker status
* Nginx status
* Deployment architecture
* Container name
* Docker image
* Port mapping
* Terraform state management

The UI is served by the Nginx container created by Terraform, demonstrating that the web content is delivered through infrastructure provisioned as code.



## Execution Evidence

The `execution-logs/` directory contains command output captured during the Terraform lifecycle, including:

* Terraform plan
* Terraform state inspection
* Docker container verification
* Terraform destroy
* Post-destroy state verification
* Post-destroy Docker verification

Screenshots are stored in the `screenshots/` directory as supporting evidence of the working infrastructure and Terraform execution.



## Key DevOps Concepts Demonstrated



### Infrastructure as Code

Infrastructure is defined declaratively in `main.tf` instead of being created manually through Docker commands.



### Declarative Configuration

The configuration describes the desired infrastructure state. Terraform determines the actions required to move the current infrastructure toward that desired state.



### Provider-Based Architecture

Terraform uses the `kreuzwerker/docker` provider to communicate with the local Docker Engine and manage Docker resources.



### Dependency Management

The container references the image resource through:

```hcl
image = docker\\\_image.nginx.image\\\_id
```

This creates a resource dependency so Terraform knows the image must be available before creating the container.



### State Management

Terraform tracks managed resources in state so it can compare the declared configuration with the infrastructure that currently exists.



### Infrastructure Lifecycle

The project demonstrates:

```text
Define → Initialize → Validate → Plan → Apply → Verify → Destroy
```

## Interview Questions \& Short Answers

### 1\. What is IaC?

Infrastructure as Code is the practice of defining and managing infrastructure using machine-readable configuration files instead of manually creating resources.

### 2\. How does Terraform work?

Terraform reads the configuration, initializes the required providers, compares the desired configuration with the current state, generates a plan, and uses providers to create, update, or destroy infrastructure.

### 3\. What is Terraform state file?

Terraform state stores information about resources managed by Terraform and helps Terraform determine what changes are required to reach the desired infrastructure state.

### 4\. Difference between `apply` and `plan`?

`terraform plan` previews proposed infrastructure changes without applying them. `terraform apply` executes the proposed changes.

### 5\. What are Terraform providers?

Providers are plugins that allow Terraform to communicate with external platforms or APIs. In this project, the Docker provider allows Terraform to manage Docker resources.

### 6\. What is resource dependency?

A resource dependency occurs when one resource requires another resource to exist first. In this project, the Docker container depends on the Docker image through its `image\\\_id` reference.

### 7\. How do you handle secret variables?

Secrets should not be hard-coded or committed to Git. They can be supplied through sensitive Terraform variables, environment variables, secret-management systems, or CI/CD secret stores.

### 8\. Explain the benefits of Terraform.

Terraform provides repeatable infrastructure, version-controlled configuration, predictable planning, automation, dependency management, and consistent infrastructure lifecycle operations.

## Cleanup

The infrastructure was destroyed after verification using:

```powershell
terraform destroy
```

This ensures that the Docker resources created for the demonstration do not remain unnecessarily active.


**Infrastructure as Code · Terraform + Docker**



