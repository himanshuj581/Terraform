
provider "aws"{
	region  = "ap-south-1"
	profile = "himanshu"
}


// Creating Key Pair
resource "tls_private_key" "myKey" {
  algorithm = "RSA"
}

module "key_pair" {
  source = "terraform-aws-modules/key-pair/aws"

  key_name   = "myKey"
  public_key = tls_private_key.myKey.public_key_openssh
}


// Creating Security Group

resource "aws_security_group" "myTask" {
  name        = "myTask"
  description = "Allow http and ssh inbound traffic"
  vpc_id      = "vpc-b6e3ffde"

  ingress {
    description = "TLS from VPC"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "TLS from VPC"
    from_port   = 22
    to_port     = 22
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
    Name = "myTask"
  }
}

// EC2 Instance

resource "aws_instance" "myInstance" {
  ami           = "ami-0447a12f28fddb066"
  instance_type = "t2.micro"
  key_name = "myKey"
  security_groups = ["myTask"]

  connection{
  		type = "ssh"
  		user = "ec2-user"
  		private_key = tls_private_key.myKey.private_key_pem
  		host = aws_instance.myInstance.public_ip
  }
  provisioner "remote-exec"{
  		inline = [
  			"sudo yum install httpd git -y",
  			"sudo systemctl restart httpd",
  			"sudo systemctl enable httpd"
  		]
  }

  tags = {
    Name = "myInstance"
  }
}


//EBS Volume
resource "aws_ebs_volume" "myVolume" {
  availability_zone = aws_instance.myInstance.availability_zone
  size              = 1

  tags = {
    Name = "myVolume"
  }
}
// volume attachement and mounting
resource "aws_volume_attachment" "ebs_instance_attachment" {
  device_name = "/dev/sdh"
  volume_id   = aws_ebs_volume.myVolume.id
  instance_id = aws_instance.myInstance.id
}



resource "null_resource" "toMount" {

  depends_on =[
  		aws_volume_attachment.ebs_instance_attachment
  	]

  connection{
  		type = "ssh"
  		user = "ec2-user"
  		private_key = tls_private_key.myKey.private_key_pem
  		host = aws_instance.myInstance.public_ip
  }

  provisioner "remote-exec"{
  	inline = [
  		"sudo mkfs.ext4 /dev/xvdh",
  		"sudo mount /dev/xvdh /var/www/html",
  		"sudo rm -rf /var/www/html/*",
  		"sudo git clone https://github.com/himanshuj581/Terraform.git /var/www/html/"
  	]
  }
 }

 //S3 bucket

 resource "aws_s3_bucket" "myBucket" {
  bucket = "task581"
  acl    = "public-read"

  tags = {
    Name        = "myBucket"
  }
}

resource "null_resource" "toDownloadFromGithub" {

  
   provisioner "local-exec"{
  	command = "git clone https://github.com/himanshuj581/image.git"
	working_dir = "C:/Users/LENOVO/Desktop/Terra/Task"
  }
 }

resource "aws_s3_bucket_object" "object" {
depends_on = [aws_s3_bucket.myBucket,
		null_resource.toDownloadFromGithub
    	     ]
  bucket = aws_s3_bucket.myBucket.bucket
  key    = "image.jpg"
  source = "C:/Users/LENOVO/Desktop/Terra/Task/image/image.jpg"
  content_type = "image/jpg"
  acl = "public-read"
}


//CloudFront
resource "aws_cloudfront_distribution" "s3_distribution" {
  origin {
    domain_name = "${aws_s3_bucket.myBucket.bucket_regional_domain_name}"
    origin_id   = "myS3Origin"

    custom_origin_config {
      http_port = 80
      https_port = 80
      origin_protocol_policy = "match-viewer"
      origin_ssl_protocols = ["TLSv1","TLSv1.1","TLSv1.2"]
    }
  }
  enabled             = true
  default_cache_behavior {
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "myS3Origin"

    forwarded_values {
      query_string = false

      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "allow-all"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  viewer_certificate{
  	cloudfront_default_certificate = true
  }
  depends_on = [
        aws_s3_bucket_object.object
    ]
 }

//Lauching of website from cmd prompt

resource "null_resource" "toLauchWebsite"  {
depends_on = [
    null_resource.toMount,
    aws_cloudfront_distribution.s3_distribution
  ]

	provisioner "local-exec" {
	    command = "start  http://${aws_instance.myInstance.public_ip}/index.html"
  	}
}
