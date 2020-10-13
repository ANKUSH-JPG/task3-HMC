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
