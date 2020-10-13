# task3-HMC

# TASK DONE:
1. Write an Infrastructure as code using Terraform, which automatically create a VPC.
2. In that VPC we have to create 2 subnets:

   a) public subnet [ Accessible for Public World!] 

   b) private subnet [ Restricted for Public World!]

3. Create a public-facing internet gateway for connecting our VPC/Network to the internet world and attach this gateway to our VPC.

4. Create a routing table for Internet gateway so that instance can connect to the outside world, update and associate it with the public subnet.

5. Launch an ec2 instance which has WordPress setup already having the security group allowing port 80 so that our client can connect to our WordPress site. Also, attach the key to an instance for further login into it.

6. Launch an ec2 instance which has MYSQL setup already with security group allowing port 3306 in a private subnet so that our WordPress VM can connect with the same. Also, attach the key with the same.

# Note: WordPress instance has to be part of public subnet so that our client can connect our site. MySQL instance has to be part of private subnet so that the outside world can't connect to it. Don't forget to add auto IP assign and auto DNS name assignment option to be enabled.


# METHOD USED:

1. Log in to your IAM account using CMD.
2. we will now create a VPC . Terraform code for the same is given below:

        resource "aws_vpc" "naitik2_vpc" {
            cidr_block = "192.168.0.0/16"
            instance_tenancy = "default"
            enable_dns_hostnames = true
            tags = {
              Name = "naitik2_vpc"
            }
          }
          
3. we need to create two subnets:
    a. public subnet [ Accessible for Public World!]
    b. private subnet [ Restricted for Public World!]
    
  The terraform code to create both the Private & the Public Subnet is as follows :
  
     resource "aws_subnet" "naitik2_public_subnet" {
            vpc_id = "${aws_vpc.naitik2_vpc.id}"
            cidr_block = "192.168.0.0/24"
            availability_zone = "ap-south-1a"
            map_public_ip_on_launch = "true"
            tags = {
              Name = "naitik2_public_subnet"
            }
          }
          
          
          
          resource "aws_subnet" "naitik2_private_subnet" {
            vpc_id = "${aws_vpc.naitik2_vpc.id}"
            cidr_block = "192.168.1.0/24"
            availability_zone = "ap-south-1a"
            tags = {
              Name = "naitik2_private_subnet"
            }
          }
          
          
4. In this step, we will be creating a public-facing internet gateway. An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.

        resource "aws_internet_gateway" "naitik2_gw" {
            vpc_id = "${aws_vpc.naitik2_vpc.id}"
            tags = {
              Name = "naitik2_gw"
            }
          }
          
5. Next, we create a Routing Table & associate it with the Public Subnet. The Terraform code for the same is stated below:

       resource "aws_route_table" "naitik2_rt" {
            vpc_id = "${aws_vpc.naitik2_vpc.id}"

            route {
              cidr_block = "0.0.0.0/0"
              gateway_id = "${aws_internet_gateway.naitik2_gw.id}"
            }

            tags = {
              Name = "naitik2_rt"
            }
          }


          resource "aws_route_table_association" "naitik2_rta" {
            subnet_id = "${aws_subnet.naitik2_public_subnet.id}"
            route_table_id = "${aws_route_table.naitik2_rt.id}"
          }
          
         
6. Now in this step we will create a security group which I will be using while launching WordPress. This security group has permissions for outside connectivity.

        resource "aws_security_group" "naitik2_sg" {

            name        = "naitik2_sg"
            vpc_id      = "${aws_vpc.naitik2_vpc.id}"


            ingress {
            
              description = "allow_http"
              from_port   = 80
              to_port     = 80
              protocol    = "tcp"
              cidr_blocks = [ "0.0.0.0/0"]

            }


               ingress {
             
               description = "allow_ssh"
               from_port   = 22
               to_port     = 22
               protocol    = "tcp"
               cidr_blocks = ["0.0.0.0/0"]
             }
             
             ingress {
             
               description = "allow_icmp"
               from_port   = 0
               to_port     = 0
               protocol    = "tcp"
               cidr_blocks = ["0.0.0.0/0"]
             }
             
             ingress {
             
               description = "allow_mysql"
               from_port   = 3306
               to_port     = 3306
               protocol    = "tcp"
               cidr_blocks = ["0.0.0.0/0"]
             }

             egress {
               from_port   = 0
               to_port     = 0
               protocol    = "-1"
               cidr_blocks = ["0.0.0.0/0"]
             }
             
             
              tags = {

              Name = "naitik2_sg"
            }
          }
          
          
 7. we will be creating one more security group, which we will be using to launch MySQL database. This security group will keep MySQL accessible only through WordPress and not through the outside world.
 
        resource "aws_security_group" "naitik2_sg_private" {

            name        = "naitik2_sg_private"
            vpc_id      = "${aws_vpc.naitik2_vpc.id}"
          
            ingress {
            
             description = "allow_mysql"
             from_port   = 3306
             to_port     = 3306
             protocol    = "tcp"
             security_groups = [aws_security_group.naitik2_sg.id]
             
           }


           ingress {
           
             description = "allow_icmp"
             from_port   = -1
             to_port     = -1
             protocol    = "icmp"
             security_groups = [aws_security_group.naitik2_sg.id]
             
           }

           egress {
           
            from_port   = 0
            to_port     = 0
            protocol    = "-1"
            cidr_blocks = ["0.0.0.0/0"]
            ipv6_cidr_blocks =  ["::/0"]
          }
          
          
          tags = {

              Name = "naitik2_sg_private"
            }
          }
          
8. launch WordPress and MySQL instances.

       resource "aws_instance" "wordpress" {
        
        ami           = "ami-ff82f990"
        instance_type = "t2.micro"
        key_name      =  "naitik2_key"
        subnet_id     = "${aws_subnet.naitik2_public_subnet.id}"
        security_groups = ["${aws_security_group.naitik2_sg.id}"]
        associate_public_ip_address = true
        availability_zone = "ap-south-1a"


        tags = {
          Name = "naitik2_wordpress"
          }
        } 
        
        resource "aws_instance" "sql" {
                    ami             =  "ami-08706cb5f68222d09"
                    instance_type   =  "t2.micro"
                    key_name        =  "naitik2_key"
                    subnet_id     = "${aws_subnet.naitik2_private_subnet.id}"
                    availability_zone = "ap-south-1a"
                    security_groups = ["${aws_security_group.naitik2_sg_private.id}"]
                    
                    tags = {
                     Name = "naitik2_sql"
                     }
                   } 
                   
               
 9. 
