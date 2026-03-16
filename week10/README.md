# Week 10: Infrastructure as Code with Terraform

## Objective

Learn the fundamentals of **Terraform** — HashiCorp's Infrastructure as Code (IaC) tool. By the end, you'll understand how to define, provision, update, and destroy infrastructure declaratively using Terraform configuration files. You'll manage real Docker containers through Terraform, working with providers, resources, variables, outputs, and state.

---

## Why Infrastructure as Code?

In Week 9 you created a Kubernetes cluster and deployed an application using CLI commands. But what happens when you need to:

- **Reproduce** the same infrastructure on another machine?
- **Track changes** to infrastructure over time?
- **Collaborate** with a team on infrastructure setup?
- **Automate** provisioning in a CI/CD pipeline?

Typing commands manually doesn't scale. **Infrastructure as Code (IaC)** solves this by defining infrastructure in configuration files that can be versioned, shared, and executed automatically.

---

## What is Terraform?

Terraform is an open-source tool that lets you define infrastructure in human-readable configuration files (HCL — HashiCorp Configuration Language), then provisions and manages that infrastructure for you.

| Feature | What It Does |
|---|---|
| Declarative configuration | You describe _what_ you want, Terraform figures out _how_ |
| Provider ecosystem | Manage AWS, Azure, GCP, Docker, Kubernetes, GitHub, and hundreds more |
| State management | Tracks what infrastructure exists so it can update or destroy it |
| Plan before apply | Preview changes before making them — no surprises |
| Idempotent | Running `apply` twice with the same config changes nothing the second time |

### How Terraform Works

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Configuration    │     │   Terraform CLI   │     │   Infrastructure  │
│  (.tf files)      │────▶│                  │────▶│   (Docker, AWS,   │
│                  │     │  init/plan/apply  │     │    K8s, etc.)     │
└──────────────────┘     └────────┬─────────┘     └──────────────────┘
                                  │
                         ┌────────▼─────────┐
                         │   State File      │
                         │  (terraform.tfstate)│
                         │                  │
                         │  Tracks what      │
                         │  exists in the    │
                         │  real world       │
                         └──────────────────┘
```

1. You write **configuration files** (`.tf`) describing the infrastructure you want
2. `terraform init` downloads the required **providers** (plugins that talk to APIs)
3. `terraform plan` compares your config to the current state and shows what will change
4. `terraform apply` makes the changes — creating, updating, or deleting resources
5. The **state file** records what Terraform created so it can manage it later

### Terraform vs. Manual Commands

| | Manual (CLI) | Terraform (IaC) |
|---|---|---|
| Reproducible | No — depends on who ran what | Yes — same config = same result |
| Trackable | No — no record of what was done | Yes — config files live in Git |
| Collaborative | Hard — "did you run that command?" | Easy — review config in a PR |
| Reversible | Manual cleanup | `terraform destroy` |
| Scalable | One resource at a time | Hundreds of resources from one file |

---

## Prerequisites

- **Docker** installed and running (Docker Desktop or Rancher Desktop)
- **Week 9** completed (familiarity with Docker and Kubernetes concepts)

---

## Project Structure (What You'll Build)

```
week-10-terraform/
├── main.tf              ← Main Terraform configuration
├── variables.tf         ← Input variable definitions
├── outputs.tf           ← Output value definitions
├── terraform.tfvars     ← Variable values (not committed to Git)
└── terraform.tfstate    ← State file (auto-generated, not committed)
```

---

## Part 1: Set Up Terraform

### Exercise 1: Install Terraform

1. Install Terraform:

   **macOS (Homebrew):**
   ```bash
   brew tap hashicorp/tap
   brew install hashicorp/tap/terraform
   ```

   **Windows (Chocolatey):**
   ```bash
   choco install terraform
   ```

   **Linux (Ubuntu/Debian):**
   ```bash
   wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
   sudo apt update && sudo apt install terraform
   ```

2. Verify the installation:
   ```bash
   terraform version
   ```

   You should see a version number (e.g., `Terraform v1.14.x`).

---

## Part 2: Your First Terraform Configuration

### Exercise 2: Create and Run a Docker Container with Terraform

Instead of running `docker run` manually, you'll tell Terraform what container you want and let it handle the rest.

1. Create a project directory:
   ```bash
   mkdir week-10-terraform && cd week-10-terraform
   ```

2. Create `main.tf`:

   ```hcl
   terraform {
     required_providers {
       docker = {
         source  = "kreuzwerker/docker"
         version = "~> 3.0"
       }
     }
   }

   provider "docker" {}

   resource "docker_image" "nginx" {
     name = "nginx:alpine"
   }

   resource "docker_container" "web" {
     name  = "terraform-nginx"
     image = docker_image.nginx.image_id

     ports {
       internal = 80
       external = 8080
     }
   }
   ```

   **What each block does:**

   | Block | Purpose |
   |---|---|
   | `terraform { required_providers }` | Declares which provider plugins Terraform needs to download |
   | `provider "docker" {}` | Configures the Docker provider (uses your local Docker daemon) |
   | `resource "docker_image" "nginx"` | Pulls the `nginx:alpine` Docker image |
   | `resource "docker_container" "web"` | Creates a container from that image, mapping port 8080 to 80 |

3. Initialize Terraform (download the Docker provider):
   ```bash
   terraform init
   ```

   You should see "Terraform has been successfully initialized!" and a `.terraform` directory is created containing the downloaded provider plugin.

4. Preview what Terraform will do:
   ```bash
   terraform plan
   ```

   Read the output carefully. Terraform shows you exactly what it will create — a `docker_image` and a `docker_container`. Nothing has been created yet.

5. Apply the configuration:
   ```bash
   terraform apply
   ```

   Terraform shows the plan again and asks for confirmation. Type `yes`.

6. Verify the container is running:
   ```bash
   docker ps
   ```

   You should see a container named `terraform-nginx`. Open http://localhost:8080 in your browser — you should see the Nginx welcome page.

7. Look at the state file:
   ```bash
   cat terraform.tfstate
   ```

   This JSON file is how Terraform tracks what it created. It maps your configuration to real infrastructure. **This file should never be committed to Git** in real projects (it can contain sensitive data).

---

## Part 3: The Terraform Workflow

### Exercise 3: Modify and Re-Apply

The core Terraform workflow is: **Write → Plan → Apply**. Let's see what happens when you change your configuration.

1. Edit `main.tf` — change the container name and add an environment variable:

   ```hcl
   resource "docker_container" "web" {
     name  = "my-web-server"
     image = docker_image.nginx.image_id

     ports {
       internal = 80
       external = 8080
     }

     env = ["NGINX_HOST=localhost"]
   }
   ```

2. Run plan to see what changes:
   ```bash
   terraform plan
   ```

   Terraform shows that it will **destroy** the old container and **create** a new one with the updated name and environment variable. Containers are immutable — changing properties requires replacement.

3. Apply the changes:
   ```bash
   terraform apply
   ```

   Type `yes`. Terraform destroys the old container and creates the new one.

4. Verify:
   ```bash
   docker ps
   ```

   You should see `my-web-server` instead of `terraform-nginx`.

### Exercise 4: Destroy Infrastructure

When you're done, Terraform can clean up everything it created.

1. Destroy all resources:
   ```bash
   terraform destroy
   ```

   Type `yes`. Terraform removes the container and the image.

2. Verify:
   ```bash
   docker ps -a | grep my-web-server
   ```

   The container is gone. This is the power of IaC — clean, complete teardown with one command.

---

## Part 4: Variables

### Exercise 5: Use Input Variables

Hardcoding values in `main.tf` isn't flexible. Variables let you parameterize your configuration.

1. Create `variables.tf`:

   ```hcl
   variable "container_name" {
     description = "Name of the Docker container"
     type        = string
     default     = "my-web-server"
   }

   variable "external_port" {
     description = "External port to map to the container"
     type        = number
     default     = 8080
   }
   ```

   **Variable block fields:**

   | Field | Purpose |
   |---|---|
   | `description` | Documents what the variable is for |
   | `type` | The data type (`string`, `number`, `bool`, `list`, `map`) |
   | `default` | Default value if none is provided (makes the variable optional) |

2. Update `main.tf` to use the variables — replace the hardcoded values in the container resource:

   ```hcl
   resource "docker_container" "web" {
     name  = var.container_name
     image = docker_image.nginx.image_id

     ports {
       internal = 80
       external = var.external_port
     }
   }
   ```

   Variables are referenced with `var.<name>`.

3. Apply with default values:
   ```bash
   terraform apply
   ```

4. Apply with custom values (override the defaults):
   ```bash
   terraform apply -var="container_name=custom-nginx" -var="external_port=9090"
   ```

   Visit http://localhost:9090 — same Nginx page, different port.

5. Clean up before the next exercise:
   ```bash
   terraform destroy
   ```

### Exercise 6: Use a Variables File

Passing `-var` flags is tedious. A `.tfvars` file is cleaner, especially for sensitive values.

1. Create `terraform.tfvars`:

   ```hcl
   container_name = "production-web"
   external_port  = 8080
   ```

   Terraform automatically loads files named `terraform.tfvars` or `*.auto.tfvars`.

2. Apply — Terraform uses the values from the file automatically:
   ```bash
   terraform apply
   ```

3. Verify:
   ```bash
   docker ps
   ```

   You should see a container named `production-web`.

   **Why use `.tfvars` files?** In real projects, `.tfvars` files often contain environment-specific values (staging vs production) or sensitive data like tokens. They are typically **gitignored**.

---

## Part 5: Outputs

### Exercise 7: Define Outputs

Outputs let you extract and display useful information after Terraform applies changes.

1. Create `outputs.tf`:

   ```hcl
   output "container_name" {
     description = "Name of the running container"
     value       = docker_container.web.name
   }

   output "container_id" {
     description = "ID of the running container"
     value       = docker_container.web.id
   }

   output "app_url" {
     description = "URL to access the application"
     value       = "http://localhost:${var.external_port}"
   }
   ```

2. Apply to see the outputs:
   ```bash
   terraform apply
   ```

   After applying, Terraform prints the output values. You can also view them anytime with:
   ```bash
   terraform output
   ```

   Outputs are useful for:
   - Displaying connection URLs after provisioning
   - Passing values between Terraform configurations
   - Providing information to CI/CD pipelines

---

## Part 6: Managing State

### Exercise 8: Understand Terraform State

Terraform state is the link between your configuration and the real world. Let's explore it.

1. View the current state:
   ```bash
   terraform show
   ```

   This shows all resources Terraform is managing, with their current attributes.

2. List the resources in state:
   ```bash
   terraform state list
   ```

   You should see:
   ```
   docker_container.web
   docker_image.nginx
   ```

3. Inspect a specific resource:
   ```bash
   terraform state show docker_container.web
   ```

   This shows every attribute of the container — name, image, ports, IP address, and more.

4. Now try something: manually delete the container outside of Terraform:
   ```bash
   docker rm -f production-web
   ```

5. Run plan:
   ```bash
   terraform plan
   ```

   Terraform detects the container is missing (state says it should exist, but it doesn't) and plans to recreate it. This is **drift detection** — Terraform reconciles the real world with your desired state.

6. Apply to restore:
   ```bash
   terraform apply
   ```

   The container is back. This is why state matters — Terraform always knows what _should_ exist and can fix discrepancies.

---

## Part 7: Multiple Resources

### Exercise 9: Manage Multiple Containers

Terraform can manage many resources at once. Let's add a second container.

1. Add another container to `main.tf` (keep everything else, add this at the bottom):

   ```hcl
   resource "docker_image" "httpd" {
     name = "httpd:alpine"
   }

   resource "docker_container" "api" {
     name  = "api-server"
     image = docker_image.httpd.image_id

     ports {
       internal = 80
       external = 9090
     }
   }
   ```

2. Plan and apply:
   ```bash
   terraform plan
   terraform apply
   ```

   Terraform only creates the new resources — it doesn't touch the existing Nginx container.

3. Verify both containers are running:
   ```bash
   docker ps
   ```

   You should see `production-web` (port 8080) and `api-server` (port 9090).

4. Visit both:
   - http://localhost:8080 — Nginx
   - http://localhost:9090 — Apache

5. Destroy everything:
   ```bash
   terraform destroy
   ```

   Both containers and both images are removed in one command.

---

## Part 8: Local-Exec Provisioners

### Exercise 10: Run Shell Commands with Terraform

Sometimes you need Terraform to run a local script — for example, to set up something that doesn't have a dedicated Terraform provider. This is what **provisioners** are for.

The `local-exec` provisioner runs a command on the machine where Terraform is running (your laptop).

1. Start fresh. Remove old files and create a new `main.tf`:

   ```bash
   rm -f main.tf variables.tf outputs.tf terraform.tfvars
   rm -f terraform.tfstate terraform.tfstate.backup
   rm -rf .terraform .terraform.lock.hcl
   ```

   ```hcl
   terraform {
     required_providers {
       null = {
         source  = "hashicorp/null"
         version = "~> 3.0"
       }
     }
   }

   variable "greeting" {
     description = "A greeting message"
     type        = string
     default     = "Hello from Terraform!"
   }

   resource "null_resource" "example" {
     provisioner "local-exec" {
       command = "echo '${var.greeting}' > message.txt && cat message.txt"
     }

     triggers = {
       greeting = var.greeting
     }
   }

   output "message" {
     value = "Check message.txt for your greeting"
   }
   ```

   **What's new here:**

   | Concept | Purpose |
   |---|---|
   | `null` provider | A special provider with no real infrastructure — useful for running scripts |
   | `null_resource` | A resource that doesn't create anything, but can run provisioners |
   | `local-exec` provisioner | Runs a shell command on your machine |
   | `triggers` | When a trigger value changes, the resource is recreated (and the provisioner runs again) |

2. Initialize and apply:
   ```bash
   terraform init
   terraform apply
   ```

   You should see your greeting echoed in the Terraform output, and a `message.txt` file is created.

3. Check the file:
   ```bash
   cat message.txt
   ```

4. Now change the greeting and re-apply:
   ```bash
   terraform apply -var='greeting=Terraform is powerful!'
   ```

   Because the `triggers` block detects the greeting changed, the `null_resource` is recreated and the command runs again with the new message.

5. Check the updated file:
   ```bash
   cat message.txt
   ```

   **Why does this matter?** In Assignment 5, you'll see `null_resource` with `local-exec` used to run a shell script that creates a Kind cluster and registers a GitHub Actions runner. The pattern is exactly the same — Terraform runs a script, and `triggers` control when it re-runs.

6. Clean up:
   ```bash
   terraform destroy
   rm -f message.txt
   ```

---

## Part 9: Clean Up

### Exercise 11: Full Cleanup

Remove all Terraform files from this exercise:

```bash
cd ..
rm -rf week-10-terraform
```

---

## Challenge Exercises

### Challenge 1: Sensitive Variables

1. Add a `sensitive` variable to your configuration:
   ```hcl
   variable "secret_key" {
     description = "A secret API key"
     type        = string
     sensitive   = true
   }
   ```
2. Use it in a `null_resource` with `local-exec` (e.g., write it to a file)
3. Create a `terraform.tfvars` with a value for `secret_key`
4. Run `terraform plan` and `terraform apply` — observe how Terraform hides the value in the output
5. Think about: why should `.tfvars` files with sensitive values be gitignored?

### Challenge 2: Use Terraform with Docker Compose-Style Setup

1. Define 3 Docker containers with Terraform: a web server, an API server, and a database (use `postgres:alpine`)
2. Use variables for all container names and ports
3. Create outputs that show the URL for each service
4. Use `terraform plan` to verify before applying
5. Scale up by duplicating a resource block — how does this compare to Kubernetes scaling?

### Challenge 3: Bridge to Assignment 5

1. Look at the Terraform files in [Assignment 5](https://github.com/arnif/assignment-5-k8s-terraform/tree/main/terraform)
2. Read `main.tf`, `variables.tf`, and `outputs.tf`
3. Identify: What provider does it use? What resource does it create? What variables does it need?
4. Compare the pattern to Exercise 10 — the structure is identical

---

## Quick Reference

### Terraform CLI Commands

| Command | Description |
|---|---|
| `terraform init` | Initialize a working directory, download providers |
| `terraform plan` | Preview changes without applying them |
| `terraform apply` | Apply changes to reach the desired state |
| `terraform destroy` | Destroy all managed infrastructure |
| `terraform fmt` | Format configuration files to canonical style |
| `terraform validate` | Check configuration for syntax errors |
| `terraform show` | Show the current state or a plan |
| `terraform output` | Display output values |
| `terraform state list` | List resources in the state |
| `terraform state show <resource>` | Show details of a specific resource |

### HCL Configuration Blocks

| Block | Purpose | Example |
|---|---|---|
| `terraform {}` | Settings, required providers | `required_providers { ... }` |
| `provider "<name>" {}` | Configure a provider | `provider "docker" {}` |
| `resource "<type>" "<name>" {}` | Define infrastructure | `resource "docker_container" "web" {}` |
| `variable "<name>" {}` | Declare an input variable | `variable "port" { type = number }` |
| `output "<name>" {}` | Declare an output value | `output "url" { value = "..." }` |

### Variable Types

| Type | Example Value |
|---|---|
| `string` | `"hello"` |
| `number` | `8080` |
| `bool` | `true` |
| `list(string)` | `["a", "b", "c"]` |
| `map(string)` | `{ key = "value" }` |

### Variable Precedence (lowest to highest)

| Source | How |
|---|---|
| Default value | `default = "value"` in variable block |
| `terraform.tfvars` file | Auto-loaded if present |
| `*.auto.tfvars` files | Auto-loaded alphabetically |
| `-var-file` flag | `terraform apply -var-file="prod.tfvars"` |
| `-var` flag | `terraform apply -var="name=value"` |
| `TF_VAR_` environment variable | `TF_VAR_name=value terraform apply` |

### Key Files

| File | Purpose | Commit to Git? |
|---|---|---|
| `*.tf` | Configuration files | Yes |
| `terraform.tfvars` | Variable values (may contain secrets) | Usually no |
| `.terraform/` | Downloaded providers and modules | No (add to `.gitignore`) |
| `terraform.tfstate` | State file (tracks real infrastructure) | No (add to `.gitignore`) |
| `.terraform.lock.hcl` | Provider version lock file | Yes |

---

## Next Steps

After completing these exercises, you're ready for:
- **Assignment 5** — Use Terraform with `null_resource` and `local-exec` to provision a Kind cluster and register a GitHub Actions self-hosted runner
- **Remote state** — Store state in a shared backend (S3, HCP Terraform) for team collaboration
- **Modules** — Reusable, shareable Terraform configurations
- **Cloud providers** — Manage AWS, Azure, or GCP infrastructure with the same workflow
