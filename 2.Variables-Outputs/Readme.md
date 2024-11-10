### **2. Variables and Outputs in Terraform**

In Terraform, variables and outputs are essential concepts for creating flexible, reusable, and secure infrastructure as code. They allow you to parameterize your configurations and expose useful information after Terraform runs. Understanding how to properly use **Input Variables**, **Output Values**, and **Sensitive Variables** is key to working effectively in a production environment.

### **2.1. Input Variables**

**Input variables** are used to make your Terraform configurations flexible by allowing values to be passed into your configuration at runtime. Instead of hardcoding values in your configuration files, you can define variables and pass in values dynamically. This makes the configuration more modular, reusable, and environment-agnostic.

#### **Why Use Input Variables?**
- **Parameterization**: You can reuse the same Terraform code for different environments (e.g., dev, staging, prod) or configurations by passing in different values.
- **Flexibility**: Input variables make your configuration more adaptable, allowing users to modify values without changing the underlying code.
- **Separation of Concerns**: They help separate the logic (your Terraform code) from the actual data (values), improving code organization.

#### **How to Define and Use Input Variables**

1. **Defining Input Variables**:
   You define input variables in a `variables.tf` (or any `.tf` file) in your Terraform configuration.

   ```hcl
   variable "instance_type" {
     description = "Type of the EC2 instance"
     type        = string
     default     = "t2.micro"
   }

   variable "region" {
     description = "AWS region to launch the resources"
     type        = string
   }
   ```

   - `description`: A short description of what the variable represents.
   - `type`: Specifies the type of the variable (e.g., `string`, `number`, `bool`, `list`, `map`). This ensures Terraform validates the value passed in.
   - `default`: An optional default value for the variable. If you don't provide a value when running Terraform, this default is used.

2. **Using Input Variables**:
   You can use variables in your Terraform resources, modules, and other places by referencing them with the `var` keyword.

   ```hcl
   resource "aws_instance" "example" {
     ami           = "ami-12345678"
     instance_type = var.instance_type
     availability_zone = var.region
   }
   ```

   In the above example, the `instance_type` and `region` values are parameterized and are fetched from the input variables.

3. **Providing Values for Input Variables**:

   There are multiple ways to provide values for input variables:

   - **`terraform.tfvars` file**: This is the most common method. You can define a `terraform.tfvars` file that contains the values for the input variables.

     ```hcl
     # terraform.tfvars
     instance_type = "t2.large"
     region        = "us-west-2"
     ```

   - **Command-line flags**: You can override variables using the `-var` flag when running Terraform commands.
     ```bash
     terraform apply -var="instance_type=t2.large" -var="region=us-west-2"
     ```

   - **Environment variables**: Terraform can read variables from environment variables prefixed with `TF_VAR_`.
     ```bash
     export TF_VAR_instance_type="t2.large"
     export TF_VAR_region="us-west-2"
     terraform apply
     ```

4. **Variable Types**:
   Terraform supports several types for input variables:
   - **String**: Simple text string.
   - **Number**: Integer or floating-point numbers.
   - **Bool**: True or False values.
   - **List**: A list of values.
   - **Map**: A key-value pair of values.
   - **Object**: A structured value with multiple named attributes.

   Example of a list and map:
   ```hcl
   variable "subnet_ids" {
     type = list(string)
   }

   variable "instance_tags" {
     type = map(string)
   }
   ```

#### **Default Values and Validation**
You can provide default values for variables, as shown above. Additionally, you can add **validation rules** to ensure that input values meet certain conditions.

Example of a variable with validation:
```hcl
variable "instance_type" {
  type        = string
  description = "Type of EC2 instance"
  default     = "t2.micro"

  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Invalid instance type. Choose from t2.micro, t2.small, or t2.medium."
  }
}
```

---

### **2.2. Output Values**

**Output values** allow you to export and display data from your Terraform-managed infrastructure after a run (i.e., after `terraform apply`). Outputs are commonly used to:
- Display information to users (e.g., public IP address of an EC2 instance).
- Pass data between Terraform modules.
- Share information with other tools and systems.

#### **Why Use Output Values?**
- **Information Sharing**: Outputs allow you to share important information from your infrastructure, such as resource IDs, URLs, or IP addresses.
- **Automation**: Outputs can be consumed by other tools, modules, or processes to automate further steps (e.g., passing a URL to a CI/CD pipeline).
- **Reusability**: By using outputs, you can expose useful values in a reusable manner across environments or projects.

#### **How to Define and Use Output Values**

1. **Defining Output Values**:
   You define output values in the `outputs.tf` file (or any `.tf` file) in your Terraform configuration.

   ```hcl
   output "instance_ip" {
     description = "The public IP address of the EC2 instance"
     value       = aws_instance.example.public_ip
   }

   output "instance_id" {
     description = "The ID of the EC2 instance"
     value       = aws_instance.example.id
   }
   ```

   - `description`: A description of what the output represents.
   - `value`: The value to be output, which can reference resource attributes.

2. **Using Output Values**:
   After you run `terraform apply`, Terraform will display the output values:

   ```bash
   terraform apply

   Outputs:
     instance_id = i-0a1b2c3d4e5f6g7h8
     instance_ip = 34.205.123.456
   ```

3. **Accessing Outputs**:
   You can reference the output values in other parts of your Terraform configuration (e.g., another module) or external systems.

   - **Accessing Outputs from Another Module**:
     If you use modules, you can expose output values from a module and then access them in the root module or other modules.

     In the **child module** (e.g., `module/ec2_instance`):
     ```hcl
     output "instance_ip" {
       value = aws_instance.example.public_ip
     }
     ```

     In the **root module**:
     ```hcl
     module "ec2_instance" {
       source = "./modules/ec2_instance"
     }

     output "instance_ip" {
       value = module.ec2_instance.instance_ip
     }
     ```

   - **Consuming Outputs in Other Systems**:
     After a Terraform apply, outputs can be consumed by automation tools like CI/CD pipelines, configuration management tools, or monitoring systems.

---

### **2.3. Sensitive Variables**

Sensitive variables are those that store sensitive data such as API keys, passwords, or tokens. These should not be displayed in logs, output, or state files to prevent accidental exposure.

#### **Why Use Sensitive Variables?**
- **Security**: Sensitive variables often contain private information (e.g., database passwords, access tokens) that you don't want exposed in the Terraform output or state files.
- **Compliance**: Sensitive information like secrets must be handled securely and shouldn't be visible in plain text in Terraform’s logs or state files.

#### **How to Use Sensitive Variables**

1. **Marking Variables as Sensitive**:
   You can mark input variables or output values as **sensitive** by setting `sensitive = true`. This ensures that Terraform will mask the output in the UI and logs.

   - **Sensitive Input Variables**:
     ```hcl
     variable "db_password" {
       description = "Database password"
       type        = string
       sensitive   = true
     }
     ```

   - **Sensitive Outputs**:
     ```hcl
     output "db_password" {
       description = "Database password"
       value       = var.db_password
       sensitive   = true
     }
     ```

2. **Example of Using Sensitive Variables in Terraform**:
   Here’s how to define a sensitive variable for a password and a sensitive output for an API key:

   ```hcl
   variable "db_password" {
     description = "The password for the database"
     type        = string
     sensitive   = true
   }

   resource "aws_secretsmanager_secret" "example" {
     name = "example-secret"
     secret_string = jsonencode({
       db_password = var.db_password
     })
   }

   output "secret_id" {
     value     = aws_secretsmanager_secret.example.id
     sensitive = true
   }
   ```

   **Note**: When `sensitive = true`, Terraform ensures that the variable or output value is **not logged**, **not shown in the Terraform plan or apply output**, and **not exposed in the state file** (though

 it is still present in the state file, it's encrypted).

---

### **Summary: Key Concepts**

- **Input Variables**: These allow you to parameterize your Terraform configurations by making values configurable at runtime. You can define them in `.tf` files and pass values through `terraform.tfvars` files, environment variables, or command-line arguments.
  
- **Output Values**: Outputs are used to display or pass important information (e.g., IP addresses, IDs) after a `terraform apply`. They can also be used to share data between modules or external systems.

- **Sensitive Variables**: Marking variables or outputs as `sensitive = true` ensures that sensitive data, like passwords or tokens, is not displayed in Terraform logs, UI, or state files.

These features help Terraform configurations become more flexible, secure, and reusable in different environments.