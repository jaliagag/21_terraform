The general syntax for creating a resource in Terraform is:


```tf
“resource "<PROVIDER>_<TYPE>" "<NAME>" {
  [CONFIG ...]
}”
```

where PROVIDER is the name of a provider (e.g., aws), TYPE is the type of resource to create in that provider (e.g., instance), NAME is an identifier you can use throughout the Terraform code to refer to this resource (e.g., my_instance), and CONFIG consists of one or more arguments that are specific to that resource.

For example, to deploy a single (virtual) server in AWS, known as an EC2 Instance, use the aws_instance resource in main.tf as follows:

```tf
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

An expression in Terraform is anything that returns a value. You’ve already seen the simplest type of expressions, literals, such as strings (e.g., "ami-0c55b159cbfafe1f0") and numbers (e.g., 5).

One particularly useful type of expression is a _reference_, which allows you to access values from other parts of your code. To access the ID of the security group resource, you are going to need to use a resource attribute reference, which uses the following syntax:

`<PROVIDER>_<TYPE>.<NAME>.<ATTRIBUTE>`

where PROVIDER is the name of the provider (e.g., aws), TYPE is the type of resource (e.g., security_group), NAME is the name of that resource (e.g., the security group is named "instance"), and ATTRIBUTE is either one of the arguments of that resource (e.g., name) or one of the attributes exported by the resource (you can find the list of available attributes in the documentation for each resource). The security group exports an attribute called id , so the expression to reference it will look like this:

`aws_security_group.instance.id`

When you add a reference from one resource to another, you create an implicit dependency. Terraform parses these dependencies, builds a dependency graph from them, and uses that to automatically determine in which order it should create resources. For example, if you were deploying this code from scratch, Terraform would know that it needs to create the security group before the EC2 Instance, because the EC2 Instance references the ID of the security group. You can even get Terraform to show you the dependency graph by running the graph command:

`$ terraform graph`

Terraform allows you to define input variables. Here’s the syntax for declaring a variable:

variable "NAME" {
  [CONFIG ...]
}

The body of the variable declaration can contain three parameters, all of them optional:

- description: It’s always a good idea to use this parameter to document how a variable is used. Your teammates will not only be able to see this description while reading the code, but also when running the plan or apply commands (you’ll see an example of this shortly).
- default: There are a number of ways to provide a value for the variable, including passing it in at the command line (using the -var option), via a file (using the -var-file option), or via an environment variable (Terraform looks for environment variables of the name TF_VAR_<variable_name>). If no value is passed in, the variable will fall back to this default value. If there is no default value, Terraform will interactively prompt the user for one.
- type: This allows you enforce type constraints on the variables a user passes in. Terraform supports a number of type constraints, including string, number, bool, list, map, set, object, tuple, and any. If you don’t specify a type, Terraform assumes the type[…]

To use the value from an input variable in your Terraform code, you can use a new type of expression called a variable reference, which has the following syntax:

var.<VARIABLE_NAME>
 w
To use a reference inside of a string literal, you need to use a new type of expression called an interpolation, which has the following syntax:

`${...}`

In addition to input variables, Terraform also allows you to define output variables by using the following syntax:

```tf
output "<NAME>" {
  value = <VALUE>
  [CONFIG ...]
}
```

The NAME is the name of the output variable, and VALUE can be any Terraform expression that you would like to output. The CONFIG can contain two additional parameters, both optional:

- description: It’s always a good idea to use this parameter to document what type of data is contained in the output variable.
- sensitive: Set this parameter to true to instruct Terraform not to log this output at the end of terraform apply. This is useful if the output variable contains sensitive material or secrets such as passwords or private keys.

For example, instead of having to manually poke around the EC2 console to find the IP address of your server, you can provide the IP address as an output variable:

```tf
output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP address of the web server"
}
```

This code uses an attribute reference again, this time referencing the public_ip attribute of the aws_instance resource. ”

output variables show up in the console after you run terraform apply, which users of your Terraform code might find useful (e.g., you now know what IP to test after the web server is deployed). You can also use the terraform output command to list all outputs without applying any changes:

```console
$ terraform output
public_ip = 54.174.13.5
```

And you can run terraform output <OUTPUT_NAME> to see the value of a specific output called <OUTPUT_NAME>:

```console
$ terraform output public_ip
54.174.13.5
```




