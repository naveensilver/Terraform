### **10. Workflows & Automation in Terraform**

In modern infrastructure management, **automation** is key. Terraform provides the ability to automate the provisioning, updating, and management of your infrastructure. Integrating Terraform with **CI/CD pipelines** and automating workflows like `terraform plan` and `terraform apply` can greatly enhance the efficiency, reliability, and repeatability of infrastructure management. This is particularly important in production environments where manual intervention can lead to errors or delays.

Let's explore **CI/CD integration** and **Terraform Plan and Apply automation** in detail.

---

### **10.1. CI/CD Integration with Terraform**

Integrating **Terraform** into a **CI/CD pipeline** allows teams to automate infrastructure provisioning and updates whenever there is a code change or new deployment. By integrating Terraform with tools like **Jenkins**, **GitLab CI**, **GitHub Actions**, or **CircleCI**, you can achieve seamless workflows for provisioning, testing, and deploying infrastructure as code.

#### **Why Integrate Terraform with CI/CD?**
- **Automated Infrastructure Provisioning**: When infrastructure changes are committed (e.g., a new resource type is added or updated), CI/CD pipelines automatically trigger Terraform to apply those changes to your infrastructure.
- **Consistency**: Using Terraform in a pipeline ensures the same infrastructure code is applied across environments (dev, staging, production) with minimal differences, ensuring consistency.
- **Version Control and Collaboration**: Changes to the infrastructure are tracked in version control (e.g., Git), enabling collaboration, rollback capabilities, and audit trails.
- **Quality Assurance**: Terraform can be integrated with automated tests (e.g., `terraform plan`) to catch errors and validate infrastructure changes before applying them.

---

#### **10.2. CI/CD Pipeline with Terraform (Example with GitHub Actions)**

GitHub Actions is a powerful CI/CD tool that allows you to automate workflows directly in GitHub. Here’s an example of how to integrate Terraform with GitHub Actions.

1. **Create a GitHub Actions Workflow File**

Create a file `.github/workflows/terraform.yml` in your repository, which will define the steps to run Terraform in your CI/CD pipeline.

```yaml
name: Terraform CI/CD Pipeline

on:
  pull_request:
    paths:
      - '**/*.tf'
  push:
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.4.6

      - name: Terraform Format Check
        run: terraform fmt -check -diff

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan -out=tfplan

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
        if: github.ref == 'refs/heads/main'  # Only apply on push to main branch

      - name: Terraform Destroy (Optional)
        run: terraform destroy -auto-approve
        if: github.event_name == 'pull_request'  # Destroy on PR to clean up resources
```

### Breakdown of the Workflow:
1. **Triggering Events**: 
   - The workflow is triggered on **push** to the `main` branch or on a **pull request** that modifies Terraform files (`.tf`).
2. **Checkout Code**: The `actions/checkout@v2` action is used to pull the latest code from the repository.
3. **Terraform Setup**: The `hashicorp/setup-terraform@v1` action installs a specified version of Terraform on the runner.
4. **Terraform Format Check**: `terraform fmt -check` ensures that all Terraform files are correctly formatted.
5. **Terraform Initialization**: The `terraform init` command initializes the working directory with the necessary Terraform providers and modules.
6. **Terraform Plan**: The `terraform plan` command generates an execution plan. This is useful to verify the changes that Terraform will apply to the infrastructure.
7. **Terraform Apply**: The `terraform apply` command automatically applies the planned changes if the `github.ref` is `main` (i.e., only in production).
8. **Terraform Destroy (Optional)**: If this is a **pull request** workflow, you may want to clean up resources after testing by using `terraform destroy`.

**Benefits**:
- **Automated Plan and Apply**: This ensures Terraform plans and applies changes to your infrastructure automatically whenever changes are pushed to the repository.
- **Environment-specific Configurations**: You can configure different pipelines for **dev**, **staging**, and **production** environments, controlling what gets applied to which environment.

---

### **10.3. Terraform Plan and Apply Automation**

While CI/CD pipelines automate Terraform actions, it's important to ensure that **Terraform Plan** and **Terraform Apply** are done in a controlled, safe, and repeatable manner, particularly for production environments.

#### **Terraform Plan Automation**

The `terraform plan` command shows you the changes Terraform will make to your infrastructure without actually applying them. This can be automated as part of the CI/CD pipeline to catch potential errors before the changes are applied.

- **Preview Changes**: By running `terraform plan` automatically, you can see what changes Terraform is proposing and get alerts on unexpected or unwanted changes.
- **Validation**: You can add automated tests to ensure that the changes Terraform plans are correct and meet your requirements before applying them.

#### **Terraform Apply Automation (Controlled)**

Automating `terraform apply` ensures that infrastructure changes are applied consistently without manual intervention, but you need to be careful in production environments.

- **Approval Gate**: In production, you can set up an approval process in the pipeline, where `terraform plan` runs and waits for manual approval before running `terraform apply`. For example, in **GitHub Actions**, you can use the `workflow_run` event or approval mechanisms like **Terraform Cloud** to enforce manual reviews.
  
Example of **manual approval** in GitHub Actions:
```yaml
jobs:
  terraform-apply:
    runs-on: ubuntu-latest
    needs: terraform-plan  # Ensure terraform plan has run successfully before apply
    steps:
      - name: Wait for approval
        run: |
          echo "Waiting for approval..."
        if: github.event_name == 'workflow_run'
      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
        if: github.ref == 'refs/heads/main'  # Only apply on push to main branch
```

---

### **10.4. Terraform Cloud/Enterprise Automation**

For teams working in larger environments or production systems, **Terraform Cloud** or **Terraform Enterprise** offers enhanced automation features, such as:

- **Remote Execution**: Terraform Cloud allows you to run Terraform commands (like `plan` and `apply`) in a secure, remote environment without needing to set up your own CI/CD pipeline.
- **State Management**: It provides state management with a **remote backend** and allows you to track state changes.
- **Collaboration**: Terraform Cloud/Enterprise supports collaboration with teams, allowing multiple users to work on the same infrastructure with role-based access controls (RBAC).
- **Approval Workflows**: It supports manual approval steps before applying infrastructure changes, which is essential for **production environments**.

---

### **10.5. Example: Automated CI/CD Workflow for Terraform on Jenkins**

Here’s a simple example of how you could set up a **Jenkins** pipeline to automate Terraform:

1. **Jenkinsfile** (Declarative Pipeline Syntax)

```groovy
pipeline {
    agent any
    environment {
        TF_VAR_region = 'us-west-2'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-org/terraform-repo.git'
            }
        }
        stage('Terraform Init') {
            steps {
                script {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                script {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }
        stage('Terraform Apply') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```

- This pipeline performs the following:
  - **Checkout Code**: Pull the latest version of the Terraform code.
  - **Terraform Init**: Initialize Terraform working directory.
  - **Terraform Plan**: Generate an execution plan.
  - **Terraform Apply**: Automatically apply the changes on the `main` branch.
  - **Cleanup**: Clean up the workspace after the pipeline runs.

---

### **10.6. Best Practices for Terraform CI/CD Automation**

- **Security**: Always secure your Terraform state file and sensitive data (e.g., using a remote backend with encryption, like AWS S3, or using Terraform Cloud).
- **Plan Approval**: In production environments, always require **manual approval** for `terraform apply` to prevent accidental or harmful changes.
- **Version Control**: Store all Terraform configurations in **version control** (e.g., GitHub, GitLab) to ensure traceability and easy rollback of changes.
- **Testing**: Automate testing of Terraform configurations, including syntax validation (`terraform fmt`), dry-run execution (`terraform plan`), and security checks.
- **Environment Isolation**: Use **workspaces** or separate Terraform configurations for different environments (e.g., dev, staging, prod).

---

### **10.7. Conclusion**

**Workflows and automation** in Terraform, through CI/CD integrations and controlled `plan

` and `apply` processes, help streamline infrastructure management. By integrating Terraform with CI/CD tools like **GitHub Actions**, **Jenkins**, and **Terraform Cloud**, you can automate infrastructure provisioning, ensure consistency, and reduce the risk of human error in production environments. Following best practices such as approval workflows, secure state management, and proper testing ensures that your infrastructure changes are automated, repeatable, and secure.