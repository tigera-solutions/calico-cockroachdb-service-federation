
1-create vpc

aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region us-east-1 --query Vpc.VpcId --output text
aws ec2 create-vpc --cidr-block 10.1.0.0/16 --region us-east-2 --query Vpc.VpcId --output text
aws ec2 create-vpc --cidr-block 10.2.0.0/16 --region us-west-1 --query Vpc.VpcId --output text


The command returns the ID of the new VPC. The following is an example.

vpc-2f09a348


vpca=
vpcb=
vpcc= 

2- create subnet
aws ec2 create-subnet --vpc-id $vpca --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text
aws ec2 create-subnet --vpc-id $vpcb --cidr-block 10.1.1.0/24 --query Subnet.SubnetId --output text
aws ec2 create-subnet --vpc-id $vpcb --cidr-block 10.2.1.0/24 --query Subnet.SubnetId --output text

subnet_a=
subnet_b=
subnet_c=

3- create internet gw

aws ec2 create-internet-gateway --region us-east-1 --query InternetGateway.InternetGatewayId --output text
aws ec2 create-internet-gateway --region us-east-2 --query InternetGateway.InternetGatewayId --output text
aws ec2 create-internet-gateway --region us-west-1 --query InternetGateway.InternetGatewayId --output text


The command returns the ID of the new VPC. The following is an example.
igw-0a5de0602dfa3257c

igwa=igw-0a5de0602dfa3257c
igwb
igwc


Using the ID from the previous step, attach the internet gateway to your VPC using the following attach-internet-gateway command.

aws ec2 attach-internet-gateway --vpc-id $vpca --internet-gateway-id $igwa
aws ec2 attach-internet-gateway --vpc-id $vpcb --internet-gateway-id $igwb
aws ec2 attach-internet-gateway --vpc-id $vpcc --internet-gateway-id $igwc


4- reate route table (VPC-A-RT-public)

aws ec2 create-route-table --vpc-id $vpca --query RouteTable.RouteTableId --output text
aws ec2 create-route-table --vpc-id $vpcb --query RouteTable.RouteTableId --output text
aws ec2 create-route-table --vpc-id $vpcc --query RouteTable.RouteTableId --output text

rtb_a=rtb-03b998aef9258873f
rtb_b
rtb_c

Create a route in the route table that points all traffic (0.0.0.0/0) to the internet gateway using the following create-route command.

aws ec2 create-route --route-table-id $rtb_a --destination-cidr-block 0.0.0.0/0 --gateway-id $igwa
aws ec2 create-route --route-table-id $rtb_b --destination-cidr-block 0.0.0.0/0 --gateway-id $igwb
aws ec2 create-route --route-table-id $rtb_c --destination-cidr-block 0.0.0.0/0 --gateway-id $igwc

Associate routing table with a subnet in your VPC so that traffic from that subnet is routed to the internet gateway.

aws ec2 associate-route-table --subnet-id $subnet_a --route-table-id $rtb_a
aws ec2 associate-route-table --subnet-id $subnet_b --route-table-id $rtb_b
aws ec2 associate-route-table --subnet-id $subnet_c --route-table-id $rtb_c
