<h1>Highload-architecture</h1>
<p>Before creating VPC I configured aws by using command <code>aws configure</code></p>
<p>AWS Access Key ID [None]: some key <br/>
AWS Secret Access Key [None]: some key <br/>
Default region name [None]: eu-west-2<br/>
Default output format [None]: json<br/></p>
<h2>Create VPC</h2>
<code>VPCID = $(aws ec2 create-vpc --cidr-block 10.10.0.0/18 --no-amazon-provided-ipv6-cidr-block --tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=kma-genesis},{Key=Lesson,Value=public-clouds}] --query Vpc.VpcId --output text)</code>
<h2>Create 3 subnets within VPC</h2>
<code>SUBNET1ID = $(aws ec2 create-subnet --vpc-id $VPCID --availability-zone eu-west-2a --cidr-block 10.10.1.0/24 --query Subnet.SubnetId --output text)</code><br/>
<code>SUBNET2ID = $(aws ec2 create-subnet --vpc-id $VPCID --availability-zone eu-west-2b --cidr-block 10.10.2.0/24 --query Subnet.SubnetId --output text)</code><br/>
<code>SUBNET3ID = $(aws ec2 create-subnet --vpc-id $VPCID --availability-zone eu-west-2c --cidr-block 10.10.3.0/24 --query Subnet.SubnetId --output text)</code><br/>
<h2>Internet Gateway</h2>
<h3>Create Internet Gateway</h3>
<code>IGWID = $(aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text)</code>
<h3>Attach Internet Gateway to VPC</h3>
<code>aws ec2 attach-internet-gateway --vpc-id $VPCID --internet-gateway-id $IGWID</code>
<h2>Security groups</h2>
<code>SGID = $(aws ec2 create-security-group --group-name my-sg --description "my security groups" --vpc-id $VPCID --output text)</code>
<p>Allow connection to TCP ports 22 (SSH), 80 (HTTP), 443 (HTTPS) for security group:</p>
<code>aws ec2 authorize-security-group-ingress --group-id $SGID --protocol tcp --port 22 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 authorize-security-group-ingress --group-id $SGID --protocol tcp --port 80 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 authorize-security-group-ingress --group-id $SGID --protocol tcp --port 443 --cidr 0.0.0.0/0</code><br/>
<h2>AWS Autoscaling group (ASG)</h2>
<h3>Create launch template</h3>
<p>Create EC2 instance within ASG based on latest AMI Amazon Linux 2 with 15GiB attached EBS</p>
<code>LTID = $(aws ec2 create-launch-template --launch-template-name my-template --version-description version1 --launch-template-data {\"NetworkInterfaces\":[{\"DeviceIndex\":0,\"AssociatePublicIpAddress\":true,\"Groups\":[\"$SGID\"],\"DeleteOnTermination\":true}],\"ImageId\":\"ami-f976839e\",\"InstanceType\":\"t2.micro\",\"TagSpecifications\":[{\"ResourceType\":\"instance\",\"Tags\":[{\"Key\":\"Name\",\"Value\":\"my-autoscaling\"}]}],\"BlockDeviceMappings\":[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"VolumeSize\":15}}]} --region eu-west-2)</code>
<h3>Create AWS Autoscaling group (ASG)</h3>
<code>aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-auto-scaling-group --launch-template "LaunchTemplateName=my-template" --min-size 1 --max-size 3 --desired-capacity 2 --vpc-zone-identifier "$SUBNET1ID,$SUBNET2ID,$SUBNET3ID" --availability-zones "eu-west-2a" "eu-west-2b" "eu-west-2c"</code>
<h2>Application Load Balancer (ALB)</h2>
<code>aws elbv2 create-load-balancer --name my-lb --subnets $SUBNET1ID $SUBNET2ID $SUBNET3ID --security-groups $SGID</code>
<h2>Delete all</h2>
<code>aws elb delete-load-balancer --load-balancer-name my-lb</code><br/>
<code>aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-auto-scaling-group</code><br/>
<code>aws ec2 delete-launch-template --launch-template-name my-lt</code><br/>
<code>aws ec2 revoke-security-group-ingress --group-id $SGID --protocol tcp --port 22 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 revoke-security-group-ingress --group-id $SGID --protocol tcp --port 80 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 revoke-security-group-ingress --group-id $SGID --protocol tcp --port 443 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 delete-security-group --group-id $SGID</code><br/>
<code>aws ec2 detach-internet-gateway --internet-gateway-id $IGWID --vpc-id $VPCID</code><br/>
<code>aws ec2 delete-internet-gateway --internet-gateway-id $IGWID</code><br/>
<code>aws ec2 delete-subnet --subnet-id $SUBNET1ID</code><br/>
<code>aws ec2 delete-subnet --subnet-id $SUBNET2ID</code><br/>
<code>aws ec2 delete-subnet --subnet-id $SUBNET3ID</code><br/>
<code>aws ec2 delete-vpc --vpc-id $VPCID</code>
