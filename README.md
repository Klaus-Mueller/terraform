# Terraform Examples: AWS & Docker on macOS

This repository contains two example projects demonstrating how to use [Terraform](https://www.terraform.io/) to provision infrastructure on AWS and Docker. It also documents key lessons and troubleshooting steps, especially for using Docker with Terraform on macOS.

---

## Project Structure

- `learn-terraform-get-started-aws/` — Example: Provisioning an EC2 instance and VPC on AWS
- `learn-terraform-docker-container/` — Example: Running an nginx container with Docker Desktop on macOS

---

## 1. AWS Example: `learn-terraform-get-started-aws/`

### Purpose
- Provisions an AWS EC2 instance running Ubuntu, inside a VPC with public and private subnets.
- Uses the official AWS provider and the `terraform-aws-modules/vpc/aws` module.

### Key Files
- `main.tf` — Main configuration (provider, AMI lookup, EC2 instance, VPC module)
- `variables.tf` — Input variables for instance name and type
- `outputs.tf` — Outputs the private DNS name of the instance

### Example Variables
```hcl
variable "instance_name" {
  description = "Value of the EC2 instance's Name tag."
  type        = string
  default     = "learn-terraform"
}

variable "instance_type" {
  description = "The EC2 instance's type."
  type        = string
  default     = "t2.micro"
}
```

### Example Output
```hcl
output "instance_hostname" {
  description = "Private DNS name of the EC2 instance."
  value       = aws_instance.app_server.private_dns
}
```

---

## 2. Docker Example: `learn-terraform-docker-container/`

### Purpose
- Provisions an nginx Docker container using the [kreuzwerker/docker](https://registry.terraform.io/providers/kreuzwerker/docker/latest) provider.
- Designed for use with Docker Desktop on macOS.

### Key Files
- `main.tf` — Main configuration (provider, nginx image, container resource)
- `variables.tf` — Input variable for container name
- `output.tf` — Outputs for container and image IDs

### Example Variable
```hcl
variable "container_name" {
  description = "Value of the name for the Docker container"
  type        = string
  default     = "ExampleNginxContainer"
}
```

### Example Outputs
```hcl
output "container_id" {
  description = "ID of the docker container"
  value       = docker_container.nginx.id
}

output "image_id" {
  description = "ID of the Docker image"
  value       = docker_image.nginx.id
}
```

---

## Lessons Learned: Docker Provider on macOS

### Problem
- The Terraform Docker provider defaults to `/var/run/docker.sock`, but Docker Desktop for Mac uses a different socket path and context system.
- This causes errors like:
  - `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`
  - `Error response from daemon: Not Found`

### Solution
- Set the `DOCKER_HOST` environment variable to the correct socket for Docker Desktop:
  ```sh
  export DOCKER_HOST=unix:///Users/$(whoami)/.docker/run/docker.sock
  ```
- Add this to your `~/.zshrc` to make it permanent.
- Always ensure Docker Desktop is running before applying Terraform.

### Additional Tips
- The Docker CLI uses contexts, but the Terraform provider does **not**. Always set `DOCKER_HOST` for Terraform.
- Check your Docker context with:
  ```sh
  docker context ls
  ```
  The active context for Docker Desktop should be `desktop-linux`.

---

## Version Control & GitHub
- To push your project to GitHub:
  ```sh
  git add .
  git commit -m "Initial commit"
  git push origin main
  ```
- If your files are not showing up on GitHub, make sure you have pushed your commits.

---

**If you encounter any issues, review the troubleshooting section or open an issue.** 