provider "aws"{
	region  = "ap-south-1"
	profile = "himanshu"
}

//VPC Creation
resource "aws_vpc" "VPC" {
  cidr_block       = "10.0.0.0/16"
  enable_dns_hostnames = "true"
  tags = {
    Name = "VPC"
  }
}

# Subnet Creation
//Public subnet for Wordpress
resource "aws_subnet" "publicSubnet" {
  vpc_id     = aws_vpc.VPC.id
  cidr_block = "10.0.0.0/24"
  availability_zone = "ap-south-1a"
  map_public_ip_on_launch = "true"
  depends_on = [aws_vpc.VPC]
  tags = {
    Name = "publicSubnet"
  }
}

//Private subnet for MySQL
resource "aws_subnet" "privateSubnet" {
  vpc_id     = aws_vpc.VPC.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "ap-south-1b"
  depends_on = [aws_vpc.VPC]
  tags = {
    Name = "privateSubnet"
  }
}

//Internet Gateway for VPC
resource "aws_internet_gateway" "InternetGateway" {
  vpc_id = aws_vpc.VPC.id
  depends_on = [aws_vpc.VPC]

  tags = {
    Name = "InternetGateway"
  }
}

//Route Table
resource "aws_route_table" "PublicRouteTable" {
  vpc_id = aws_vpc.VPC.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.InternetGateway.id
  }
   depends_on = [aws_vpc.VPC, aws_internet_gateway.InternetGateway]

  tags = {
    Name = "PublicRouteTable"
  }
}


#Associate with subnet
resource "aws_route_table_association" "PublicAssociation" {
  subnet_id      = aws_subnet.publicSubnet.id
  route_table_id = aws_route_table.PublicRouteTable.id


  depends_on = [
    aws_subnet.publicSubnet ,aws_route_table.PublicRouteTable
  ]
}

#Security Group

// For Wordpress
resource "aws_security_group" "wordpressSecurityGroup" {
  name        = "wordpressSecurityGroup"
  description = "allows ssh and http"
  vpc_id      = aws_vpc.VPC.id


  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


    ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port = -1
    to_port = -1
    protocol  = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }


  depends_on = [ aws_vpc.VPC ]


  tags = {
    Name = "wordpressSecurityGroup"
  }
}

// For MySQL
resource "aws_security_group" "MySQLSecurityGroup" {
  name        = "MySQLSecurityGroup"
  description = "Allow only wordpress"
  vpc_id      = aws_vpc.VPC.id


  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["${aws_subnet.publicSubnet.cidr_block}"]
  }

    ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["${aws_subnet.publicSubnet.cidr_block}"]
  }

  ingress {
    from_port = -1
    to_port = -1
    protocol  = "icmp"
    cidr_blocks = ["${aws_subnet.publicSubnet.cidr_block}"]
  }
  
   egress {
    from_port   = 0
    to_port     = 0
    protocol    = -1
    cidr_blocks = ["0.0.0.0/0"]
  }  


  depends_on = [
    aws_vpc.VPC,
    aws_security_group.wordpressSecurityGroup,
  ]

  tags = {
    Name = "MySQLSecurityGroup"
  }
}

#EC2 Instances

//Wordpress Instance
resource "aws_instance" "WordPressInstance" {
  ami           = "ami-000cbce3e1b899ebd"
  instance_type = "t2.micro"
  key_name      = "key12345"
  subnet_id     = "${aws_subnet.publicSubnet.id}" 
  vpc_security_group_ids = ["${aws_security_group.wordpressSecurityGroup.id}"]
  tags = {
    Name = "WordPressInstance"
  }
}

#MySQL Instance
resource "aws_instance" "MySQLInstance" {
  ami           = "ami-08706cb5f68222d09"
  instance_type = "t2.micro"
  key_name      = "key12345"
  subnet_id     = "${aws_subnet.privateSubnet.id}"
  vpc_security_group_ids = ["${aws_security_group.MySQLSecurityGroup.id}"]
  tags = {
    Name = "MySQLInstance"
  }
}
