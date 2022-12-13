
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

3- Create internet gw

```bash
aws ec2 create-internet-gateway --region us-east-1 --query InternetGateway.InternetGatewayId --output text |read igw_a
aws ec2 create-internet-gateway --region us-east-2 --query InternetGateway.InternetGatewayId --output text |read igw_b
aws ec2 create-internet-gateway --region us-west-1 --query InternetGateway.InternetGatewayId --output text |read igw_c
```

4- Attach the internet gateway to your VPC using the following attach-internet-gateway command.
```bash
aws ec2 attach-internet-gateway --region us-east-1 --vpc-id $vpc_a --internet-gateway-id $igw_a
aws ec2 attach-internet-gateway --region us-east-2 --vpc-id $vpc_b --internet-gateway-id $igw_b
aws ec2 attach-internet-gateway --region us-west-1 --vpc-id $vpc_c --internet-gateway-id $igw_c
```

5- Create route table 

```bash
aws ec2 create-route-table --region us-east-1 --vpc-id $vpc_a --query RouteTable.RouteTableId --output text |read rtb_a
aws ec2 create-route-table --region us-east-2 --vpc-id $vpc_b --query RouteTable.RouteTableId --output text |read rtb_b
aws ec2 create-route-table --region us-west-1 --vpc-id $vpc_c --query RouteTable.RouteTableId --output text |read rtb_c
```

6- Create a route in the route table that points all traffic (0.0.0.0/0) to the internet gateway using the following create-route command.
```bash
aws ec2 create-route --region us-east-1 --route-table-id $rtb_a --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_a
aws ec2 create-route --region us-east-2 --route-table-id $rtb_b --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_b
aws ec2 create-route --region us-west-1 --route-table-id $rtb_c --destination-cidr-block 0.0.0.0/0 --gateway-id $igw_c
```

7- Associate routing table with a subnet in your VPC so that traffic from that subnet is routed to the internet gateway.
```bash
aws ec2 associate-route-table --region us-east-1 --subnet-id $subnet_a --route-table-id $rtb_a
aws ec2 associate-route-table --region us-east-2 --subnet-id $subnet_b --route-table-id $rtb_b
aws ec2 associate-route-table --region us-west-1 --subnet-id $subnet_c --route-table-id $rtb_c
```
