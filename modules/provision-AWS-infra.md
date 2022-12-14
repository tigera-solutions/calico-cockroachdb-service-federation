# Provision AWS Infrastracture

**Create VPCs and subnets** 

1- Create vpc and save the vpc id to a virable 

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region us-east-1 --query Vpc.VpcId --output text | read vpc_a
aws ec2 create-vpc --cidr-block 10.1.0.0/16 --region us-east-2 --query Vpc.VpcId --output text | read vpc_b
aws ec2 create-vpc --cidr-block 10.2.0.0/16 --region us-west-1 --query Vpc.VpcId --output text | read vpc_c
```


2- Create subnet and save subnet id to a virable

```bash
aws ec2 create-subnet --region us-east-1 --vpc-id $vpc_a --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text | read subnet_a
aws ec2 create-subnet --region us-east-2 --vpc-id $vpc_b --cidr-block 10.1.1.0/24 --query Subnet.SubnetId --output text | read subnet_b
aws ec2 create-subnet --region us-west-1 --vpc-id $vpc_c --cidr-block 10.2.1.0/24 --query Subnet.SubnetId --output text | read subnet_c
```

3- You can modify the public IP addressing behavior of your subnet so that an instance launched into the subnet automatically receives a public IP address using the following modify-subnet-attribute command

```bash
aws ec2 modify-subnet-attribute --region us-east-1 --subnet-id $subnet_a --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --region us-east-2 --subnet-id $subnet_b --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --region us-west-1 --subnet-id $subnet_c --map-public-ip-on-launch
```

4- Create internet gw

```bash
aws ec2 create-internet-gateway --region us-east-1 --query InternetGateway.InternetGatewayId --output text | read igw_a
aws ec2 create-internet-gateway --region us-east-2 --query InternetGateway.InternetGatewayId --output text | read igw_b
aws ec2 create-internet-gateway --region us-west-1 --query InternetGateway.InternetGatewayId --output text | read igw_c
```

5- Attach the internet gateway to your VPC using the following attach-internet-gateway command.
```bash
aws ec2 attach-internet-gateway --region us-east-1 --vpc-id $vpc_a --internet-gateway-id $igw_a
aws ec2 attach-internet-gateway --region us-east-2 --vpc-id $vpc_b --internet-gateway-id $igw_b
aws ec2 attach-internet-gateway --region us-west-1 --vpc-id $vpc_c --internet-gateway-id $igw_c
```

6- Create route table 

```bash
aws ec2 create-route-table --region us-east-1 --vpc-id $vpc_a --query RouteTable.RouteTableId --output text | read rtb_a
aws ec2 create-route-table --region us-east-2 --vpc-id $vpc_b --query RouteTable.RouteTableId --output text | read rtb_b
aws ec2 create-route-table --region us-west-1 --vpc-id $vpc_c --query RouteTable.RouteTableId --output text | read rtb_c
```

7- Create a route in the route table that points all traffic (0.0.0.0/0) to the internet gateway using the following create-route command.
```bash
aws ec2 create-route --region us-east-1 --route-table-id $rtb_a --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_a
aws ec2 create-route --region us-east-2 --route-table-id $rtb_b --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_b
aws ec2 create-route --region us-west-1 --route-table-id $rtb_c --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_c
```

8- Associate routing table with a subnet in your VPC so that traffic from that subnet is routed to the internet gateway.
```bash
aws ec2 associate-route-table --region us-east-1 --subnet-id $subnet_a --route-table-id $rtb_a
aws ec2 associate-route-table --region us-east-2 --subnet-id $subnet_b --route-table-id $rtb_b
aws ec2 associate-route-table --region us-west-1 --subnet-id $subnet_c --route-table-id $rtb_c
```

**Create VPC peering connection** 

1- Create VPC peeering requests and save VpcPeeringConnectionId to a variable

```bash
aws ec2 create-vpc-peering-connection --region us-east-1 --vpc-id $vpc_a --peer-vpc-id $vpc_b --peer-region us-east-2 --query VpcPeeringConnection.VpcPeeringConnectionId --output text | read vpc_a_vpc_b_peering
aws ec2 create-vpc-peering-connection --region us-east-1 --vpc-id $vpc_a --peer-vpc-id $vpc_c --peer-region us-west-1 --query VpcPeeringConnection.VpcPeeringConnectionId --output text | read vpc_a_vpc_c_peering
aws ec2 create-vpc-peering-connection --region us-east-2 --vpc-id $vpc_b --peer-vpc-id $vpc_c --peer-region us-west-1 --query VpcPeeringConnection.VpcPeeringConnectionId --output text | read vpc_b_vpc_c_peering
```

2- Using the VpcPeeringConnectionId from the previous step, accept the VPC peering connections.

```bash
aws ec2 accept-vpc-peering-connection --region us-east-2 --vpc-peering-connection-id $vpc_a_vpc_b_peering
aws ec2 accept-vpc-peering-connection --region us-west-1 --vpc-peering-connection-id $vpc_a_vpc_c_peering
aws ec2 accept-vpc-peering-connection --region us-west-1 --vpc-peering-connection-id $vpc_b_vpc_c_peering
```

3- Add routes in each routing table 

```bash
aws ec2 create-route --region us-east-1 --route-table-id $rtb_a --destination-cidr-block 10.1.1.0/24 --vpc-peering-connection-id $vpc_a_vpc_b_peering
aws ec2 create-route --region us-east-1 --route-table-id $rtb_a --destination-cidr-block 10.2.1.0/24 --vpc-peering-connection-id $vpc_a_vpc_c_peering
aws ec2 create-route --region us-east-2 --route-table-id $rtb_b --destination-cidr-block 10.0.1.0/24 --vpc-peering-connection-id $vpc_a_vpc_b_peering
aws ec2 create-route --region us-east-2 --route-table-id $rtb_b --destination-cidr-block 10.2.1.0/24 --vpc-peering-connection-id $vpc_b_vpc_c_peering
aws ec2 create-route --region us-west-1 --route-table-id $rtb_c --destination-cidr-block 10.0.1.0/24 --vpc-peering-connection-id $vpc_a_vpc_c_peering
aws ec2 create-route --region us-west-1 --route-table-id $rtb_c --destination-cidr-block 10.1.1.0/24 --vpc-peering-connection-id $vpc_b_vpc_c_peering
```

**Launch an instance into our subnets** 

1- Create a key pair and use the --query option and the --output text option to pipe your private key directly into a file with the .pem extension.

```bash
aws ec2 create-key-pair --region us-east-1 --key-name MyKeyPaira --query "KeyMaterial" --output text > MyKeyPaira.pem
aws ec2 create-key-pair --region us-east-2 --key-name MyKeyPairb --query "KeyMaterial" --output text > MyKeyPairb.pem
aws ec2 create-key-pair --region us-west-1 --key-name MyKeyPairc --query "KeyMaterial" --output text > MyKeyPairc.pem
chmod 400 MyKeyPaira.pem
chmod 400 MyKeyPairb.pem
chmod 400 MyKeyPairc.pem
```

2- Create a security group in your VPC using the create-security-group command.

```bash
aws ec2 create-security-group --region us-east-1 --group-name allow-all --description "Security group for any traffic" --vpc-id $vpc_a --query GroupId --output text | read sg_a
aws ec2 authorize-security-group-ingress --region us-east-1 --group-id $sg_a --protocol all --port all --cidr 0.0.0.0/0
aws ec2 create-security-group --region us-east-2 --group-name allow-all --description "Security group for any traffic" --vpc-id $vpc_b --query GroupId --output text | read sg_b
aws ec2 authorize-security-group-ingress --region us-east-2 --group-id $sg_b --protocol all --port all --cidr 0.0.0.0/0
aws ec2 create-security-group --region us-west-1 --group-name allow-all --description "Security group for any traffic" --vpc-id $vpc_c --query GroupId --output text | read sg_c
aws ec2 authorize-security-group-ingress --region us-west-1 --group-id $sg_c --protocol all --port all --cidr 0.0.0.0/0
```

3- Launch 3 instances into each region, using the security group and key pair you've created. 

```bash
aws ec2 run-instances --region us-east-1 --image-id ami-0bc7c87c6aee5963e --count 3 --instance-type m5.large --key-name MyKeyPaira --security-group-ids $sg_a --subnet-id $subnet_a
aws ec2 run-instances --region us-east-2 --image-id ami-075c22873590ff2e0 --count 3 --instance-type m5.large --key-name MyKeyPairb --security-group-ids $sg_b --subnet-id $subnet_b
aws ec2 run-instances --region us-west-1 --image-id ami-0589688e8c6693946 --count 3 --instance-type m5.large --key-name MyKeyPairc --security-group-ids $sg_c --subnet-id $subnet_c
```

4- When your instance is in the running state, you can connect to it using an SSH client on a Linux or Mac OS X computer by using the following command:

```bash
ssh -i "MyKeyPair.pem" ec2-user@instance-public-ip
```
