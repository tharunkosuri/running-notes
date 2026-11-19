ansible is for configuration management. ansible is not great to update and delete the infra



IaaC tools like terraform follows declarative way of creating infra

1\. version control infra

2\. reusable code called modules

3\. CRUD

4\. Inventory management

5\. Cost effective

6\. Dependency management

7\. consistent infra



providers -> a plugin that let terraform to connect with platform API to CRUD the resources



everything is block in terraform.



resource, variable, terraform, provider, output, data, etc etc.



resource "type\_of\_resource" "name\_you\_give" {

&#x09;key = "value"

}



terraform init -> downloads provider file into .terraform folder

terraform plan -> reads tf files and gives us what is going to do

terraform apply -> applies the infra

terraform destroy -> delete the infra



variable "variable\_name" {

&#x09;type = string/number/map/boolean/list

&#x09;default = "default\_value"

}



default, \*.tfvars, command line, env variables, prompt



variables.tf

terraform.tfvars/dev.tfvars

\--> by default it will take from terraform.tfvars

\--> but if we want to give explicitly the file name then (terraform plan/apply -var-file=dev.tfvars)



terraform plan/apply -var-file=dev.tfvars

env variables



export TF\_VAR\_instance\_type="t3.micro"  --> in linux

set TF\_VAR\_<var\_name>="value" --> in windows



1\. default values

2\. terraform.tfvars or \*.tfvars

3\. env variables

4\. command line

5\. prompt

========================== Validation ===========================

variable "instance\_type" {

&#x20; type        = string

&#x20; #default     = "t3.micro"

&#x20; description = "This is the instance type for web application"



&#x20; **validation** {

&#x20;   condition = contains(\["t3.micro", "t3.small", "t3.medium"], var.instance\_type)

&#x20;   error\_message = "The instance type must be one of :: "t3.micro", "t3.small", "t3.medium" "

&#x20; }

}

===================== End Validation ============================



default -> t3.micro

terraform.tfvars -> t3.small

env variable -> t3.medium

command line -> t3.large



prefernce

===========

1\. command line

2\. tfvars

3\. env variable

4\. default

5\. prompt



conditions

===========

key = expression ? "true\_value" : "false\_value"



PROD -> t3.large

DEV -> t3.micro



==================== Conditions =================



resource "aws\_instance" "conditions" {

&#x20; ami                    = "ami-0055977a6e828a4f4"

&#x20; **instance\_type          = var.environment == "dev" ? "t3.micro" : "t3.medium"**

&#x20; vpc\_security\_group\_ids = \[aws\_security\_group.conditions.id]



&#x20; tags = {

&#x20;   Name = "roboshop-conditions"

&#x20; }



}



==================== End Conditions ==================



loops

======

1\. count based loop

2\. for\_each

3\. dynamic



instance\_name = roboshop-dev-mongodb

SG\_names = roboshop-dev-mongodb, roboshop-dev-common



interpolation -> you can mix normal text with variable value in double quotes

===========

"${var.project}-${var.environment}-common"



count based loop

================

count = 10

count.index -> 0,1,2,3...9



aws\_instance.roboshop\[9]



aws\_instance.roboshop



R53, Ec2, SG



SG, EC2, R53



functions

=============

in terraform we cant create custom functions. we can only terraform functions



takes input and gives output



length(list) -> calculate number of items in the list and give us the count



list of items -> do you have apple in your list ? yes : no



fruits = \["apple","banana","pomo","guava","grapes"]



contains(fruites, "watermelon") -> true

contains(fruites, "pomo") -> false



index(fruits, "banana") -> 1

index(fruits, "berries") -> negative value



FUN\_NAME(inputs) -> output





=============================

We have different types of functions in terraform

FUN\_NAME(inputs) -> output



count = length(var.instance) # length function

count = contains(var.instance, frontend) ? 1 : 0 # contains function

records = \[aws\_instance.roboshop\[index(var.instance, "frontend")].public\_ip] # index function

max(1,7,9)

min(8,9,0)

join(" ",\["Tharun", "kumar", "Kosuri"])------------> Join Function is used to convert from string to list

output: Tharun Kumar Kosuri

join("-",\["Tharun", "kumar", "Kosuri"])

output: Tharun-Kumar-Kosuri



Split(",", "Tharun,kumar,ksouri)  --> Split function converts from string to list

\[

"Tharun",

"Kumar",

"kosuri",

]



merge(map1,map2,.....) --> Merge is used to merge multiple maps

Note: map2 appends to map1 but if there is any common values in both the maps then map2 overrided the common values in map1

\--> This is mostly used in tags



lookup(aws\_instance.roboshop, "frontend").public\_ip

\--> lookup function is used for map where to search for particular key in map

===========================================



resource "aws\_route53\_record" "roboshop" {

&#x20; #count = 3

&#x20; count = length(var.instance)

&#x20; zone\_id = var.zone\_id

&#x20; name    = "${var.instance\[count.index]}-${var.domain\_name}"

&#x20; type    = "A"

&#x20; ttl     = 1

&#x20; records = \[aws\_instance.roboshop\[count.index].private\_ip]

&#x20; allow\_overwrite = true # id records already present then it will not create the new records

}

