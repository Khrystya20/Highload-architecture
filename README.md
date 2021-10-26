<h1>Highload-architecture</h1>
<p>Before creating VPC I configured aws by using command <code>aws configure</code></p>
<p>AWS Access Key ID [None]: some key <br/>
AWS Secret Access Key [None]: some key <br/>
Default region name [None]: eu-west-2<br/>
Default output format [None]: json<br/></p>
<h2>Create VPC</h2>
<code>aws ec2 create-vpc --cidr-block 10.10.0.0/18 --no-amazon-provided-ipv6-cidr-block --tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=kma-genesis},{Key=Lesson,Value=public-clouds}] --query Vpc.VpcId --output text</code>
<p>From output:<br/> vpc id = vpc-064414e68090df6a8<br/>To see created vpc I wrote command:<br/><code>aws ec2 describe-vpcs --filters "Name=vpc-id,Values=vpc-064414e68090df6a8"</code></p>
<h2>Create 3 subnets within VPC</h2>
<p>Using created VPC I create 3 subnets, each dedicated to availability zone within region eu-west-2 (region that I chose previously in aws configuration). In next commands I use specific vpc-id (got from created vpc), availability-zone (got from chosen region) and cidr-block</p>
<code>aws ec2 create-subnet --vpc-id vpc-064414e68090df6a8 --availability-zone eu-west-2a --cidr-block 10.10.1.0/24</code><br/>
<code>aws ec2 create-subnet --vpc-id vpc-064414e68090df6a8 --availability-zone eu-west-2b --cidr-block 10.10.2.0/24</code><br/>
<code>aws ec2 create-subnet --vpc-id vpc-064414e68090df6a8 --availability-zone eu-west-2c --cidr-block 10.10.3.0/24</code><br/>
<p>From output:<br/>
   subnet-1 id = subnet-0436eef3ee9143f84<br/>
   subnet-2 id = subnet-07322446341e76b46<br/>
   subnet-3 id = subnet-0b3fb48a6c7bc654b<br/>
   To see created subnets I wrote command: <br/><code>aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-064414e68090df6a8"</code></p>
<h2>Internet Gateway</h2>
<h3>Create Internet Gateway</h3>
<code>aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text</code>
<p>From output:<br/> internet-gateway id = igw-079c90c6ad05f2df2<br/>
To see created Internet Gateway I wrote command:<br/><code>aws ec2 describe-internet-gateways --filters "Name=internet-gateway-id,Values=igw-079c90c6ad05f2df2"</code></p>
<h3>Attach Internet Gateway to VPC</h3>
<p>In next command I used specific vpc-id (got from created vpc) and internet-gateway id (got from created internet gateway)</p>
<code>aws ec2 attach-internet-gateway --vpc-id vpc-064414e68090df6a8 --internet-gateway-id igw-079c90c6ad05f2df2</code>
<h2>Security groups</h2>
<p>In next command I used specific vpc-id (got from created vpc)</p>
<code>aws ec2 create-security-group --group-name my-sg --description "my security groups" --vpc-id vpc-064414e68090df6a8</code>
<p>From output:<br/> security-group id = sg-0403591e27e43a2e5</p>
<p>Allow connection to TCP ports 22 (SSH), 80 (HTTP), 443 (HTTPS) for security group. In next commands I use specific security-group id (got from created security group)</p>
<code>aws ec2 authorize-security-group-ingress --group-id sg-0403591e27e43a2e5 --protocol tcp --port 22 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 authorize-security-group-ingress --group-id sg-0403591e27e43a2e5 --protocol tcp --port 80 --cidr 0.0.0.0/0</code><br/>
<code>aws ec2 authorize-security-group-ingress --group-id sg-0403591e27e43a2e5 --protocol tcp --port 443 --cidr 0.0.0.0/0</code><br/>
<p>To see created security groups I wrote command:<br/><code>aws ec2 describe-security-groups --filters "Name=vpc-id,Values=vpc-064414e68090df6a8"</code></p>
<h2>AWS Autoscaling group (ASG)</h2>
<h3>Create launch template</h3>
<p>Create EC2 instance within ASG based on latest AMI Amazon Linux 2 with 15GiB attached EBS</p>
<code>aws ec2 create-launch-template --launch-template-name my-template --version-description version1 --launch-template-data {\"NetworkInterfaces\":[{\"DeviceIndex\":0,\"AssociatePublicIpAddress\":true,\"Groups\":[\"sg-0403591e27e43a2e5\"],\"DeleteOnTermination\":true}],\"ImageId\":\"ami-f976839e\",\"InstanceType\":\"t2.micro\",\"TagSpecifications\":[{\"ResourceType\":\"instance\",\"Tags\":[{\"Key\":\"Name\",\"Value\":\"my-autoscaling\"}]}],\"BlockDeviceMappings\":[{\"DeviceName\":\"/dev/sda1\",\"Ebs\":{\"VolumeSize\":15}}]} --region eu-west-2</code>
<p>In this command I used specific security-group id (got from created security group), region (chosen previously) and image id according to this region</p>
<p>From output:<br/> launch-template id = lt-07c4f476b49f6baa0<br/> 
To see created launch template I wrote command:<br/><code>aws ec2 describe-launch-templates --launch-template-ids lt-07c4f476b49f6baa0</code></p>
<h3>Create AWS Autoscaling group (ASG)</h3>
<code>aws autoscaling create-auto-scaling-group --auto-scaling-group-name my-auto-scaling-group --launch-template "LaunchTemplateName=my-template" --min-size 1 --max-size 3 --desired-capacity 2 --vpc-zone-identifier "subnet-0436eef3ee9143f84,subnet-07322446341e76b46,subnet-0b3fb48a6c7bc654b" --availability-zones "eu-west-2a" "eu-west-2b" "eu-west-2c"</code>
<p>In this command I used specific subnet ids (got from created subnets) and availability zones (got from chosen region)</p>
<h2>Application Load Balancer (ALB)</h2>
<code>aws elbv2 create-load-balancer --name my-lb --subnets subnet-0436eef3ee9143f84 subnet-07322446341e76b46 subnet-0b3fb48a6c7bc654b --security-groups sg-0403591e27e43a2e5</code>
<p>In this command I used specific subnet ids (got from created subnets) and security-group id (got from created security group)</p>
