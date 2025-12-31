# Terraform Fundamentals (Variables, Locals, Data Types, Additional Functions): 12/30/25

Notes covering variables and associated concepts of parameterization; locals and associated concepts of readability and DRY principles; data types and conversions; and related functions. This class is critical preparation for modules, working in multiple environments, improving code readability, and more.

## Repo Layout
- [Syntax Demo](./syntax-demo/): This contains no providers and serves as a starting basis for exploring these features in Terraform. Of course, no AWS resources will be deployed with this.
- [AWS Infrastructure Code](./simple-code-no-refactor/): We will use this code as a starting block to refactor later on.
- [Refactored Code](./simple/): This code is my personal refactored code and one of the goals for class. Please note this code may not follow some best practices for variables (such as overuse), as utilization was prioritized.
- [For Later](./advanced/): This is the [refactored code](./simple/) with additional Terraform features added that will not be covered tonight and is for my notes. No guarantees it works properly.

Grab these folders:
```bash
# navigate into your project folder 
# for example, if you use theowaf folder, do something like this:
# cd ~/Documents/TheoWAF/class7/AWS/Terraform
# no need to create a folder of today's date

git clone --no-checkout https://github.com/aaron-dm-mcdonald/Class7-notes.git
cd Class7-notes

git sparse-checkout init --cone
git sparse-checkout set 123025/syntax-demo 123025/simple-code-no-refactor

git checkout
mv 123025/ ..
cd ..
rm -rf Class7-notes/

```

---

## Table of Contents

1. [Terraform’s evaluation model](#1-terraforms-evaluation-model-important-context)
2. [Variables vs locals](#2-variables-vs-locals)
3. [Variable types](#3-variable-types)
4. [Defining variable values at runtime](#4-defining-variable-values-at-runtime)
5. [Outputs and accessing data](#5-outputs-and-accessing-data)
6. [Conditionals (ternary operator)](#6-conditionals-ternary-operator)
7. [Splat operator](#7-splat-operator-)
8. [Arithmetic expressions](#8-arithmetic-expressions)
9. [Common functions](#9-common-functions)
10. [Type conversions](#10-type-conversions)
11. [Variable validations](#11-variable-validations)

---

## 1. Terraform’s evaluation model (important context)

Terraform evaluates **expressions**. Everything in Terraform that is provided to an argument is an **expression**.

Expressions include:
- literals (`"Hello"`, `20`, etc.)
- dot references (`aws_instance.example.subnet_id`, for example)
- functions (such as the `file()` function)
- arithmetic (`20 + 5`)

Essentially, this means in almost all situations, the right side of the equals sign is an **expression**, and during execution plan generation or provisioning of infrastructure, they are evaluated to a **value** (which has a data type that will be discussed later). This is a critical concept to understand.

Finally, note that while everything is an **expression**, not all **expressions** are allowed in certain places. Sometimes functions are not permitted, or dot references are not permitted, as examples (we will discuss one example later on).

Docs:  
https://developer.hashicorp.com/terraform/language/expressions

---

## 2. Variables vs locals

Before we continue, we must clarify two things for (likely) two sets of students:

### 1) Students not familiar with CS terminology or who have not used a scripting/programming language before
- **Parameters**: Simplified as much as possible in this context, a parameter is something that can be changed depending on the situation.
- We use parameterization to try to prevent people who are not DevOps professionals from modifying our Terraform code, or at least to streamline this process.
- This prevents minor errors, speeds deployments, and allows different teams to deploy infrastructure more independently, while encouraging them to avoid introducing state drift via ClickOps.

### 2) Students familiar with function arguments and variables in other languages
- Please forgive HashiCorp for the poor naming conventions.
- Variables are used essentially like argument parameters for functions.
- Locals are essentially used like variables in programming languages.
- There are GitHub issues on this; it seems that locals and variables were one language feature many years ago and got split up, received additional features, and the names stuck.

### Variables: inputs to a module (or just "Terraform" for now)

Variables allow callers (users of Terraform) to customize behavior. We call the values they change parameters, and the process itself is parameterization.

For example, we have a deployment with an ASG and two environments. The dev environment is often broken or wastefully used by developers testing things. It does not need enterprise-grade compute or memory. To optimize the speed of deployment and cost, we use `t3.micro` instances for the ASG. In production, we use `g5.2xlarge` instances.

By setting a variable and replacing the `aws_instance` `instance_type` argument’s **expression** from something like a literal (`"t2.micro"`) to `var.instance_type`, and adding the below code, when we run `terraform apply`, Terraform will ask us to provide an instance type.

```hcl
variable "instance_type" {
  description = "ASG instance type"
  type        = string
}
