---
comments: true
---

# **Configuring a Highly Available and Secure Application on AWS with Terraform**

Learn to create a robust AWS infrastructure using Terraform that includes an Auto Scaling Group (ASG), Application Load Balancer (ALB), security groups, launch templates, and a Web Application Firewall (WAF) within a custom VPC, and DNS configuration with Route 53.

## Prerequisites

Before starting, ensure you have the following:

- Terraform installed
- An AWS account with appropriate permissions
- AWS CLI with configured credentials
- A basic understanding of AWS and Terraform
- Code editor (VS Code)

## Step 1: AWS Provider Setup & terraform block

Define your AWS provider with the desired region and specify the required providers for Terraform.

``` hcl
provider "aws" {
  region = "ap-northeast-2"
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```

## Step 2: Create the VPC

Define your Virtual Private Cloud (VPC) with a CIDR block and tags.

``` hcl
resource "aws_vpc" "app1" {
  cidr_block = "10.74.0.0/16"

  tags = {
    Name = "app1"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Mustafar"
  }
}
```

## Step 3: Set Up Subnets

Create subnets within your VPC. Subnets can be public or private and are typically distributed across multiple availability zones for high availability. 

``` hcl
#These are   for  public

resource "aws_subnet" "public-ap-northeast-2a" {
  vpc_id                  = aws_vpc.app1.id
  cidr_block              = "10.74.1.0/24"
  availability_zone       = "ap-northeast-2a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-ca-central-1a"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Musafar"
  }
}

resource "aws_subnet" "public-ap-northeast-2d" {
  vpc_id                  = aws_vpc.app1.id
  cidr_block              = "10.74.4.0/24"
  availability_zone       = "ap-northeast-2d"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-ca-central-1b"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Musafar"
  }
}

resource "aws_subnet" "public-ap-northeast-2c" {
  vpc_id                  = aws_vpc.app1.id
  cidr_block              = "10.74.3.0/24"
  availability_zone       = "ap-northeast-2c"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-ca-central-1c"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Musafar"
  }
}



#these are for private
resource "aws_subnet" "private-ap-northeast-2a" {
  vpc_id            = aws_vpc.app1.id
  cidr_block        = "10.74.11.0/24"
  availability_zone = "ap-northeast-2a"

  tags = {
    Name = "private-ca-central-1a"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Musafar"
  }
}

resource "aws_subnet" "private-ap-northeast-2d" {
  vpc_id            = aws_vpc.app1.id
  cidr_block        = "10.74.14.0/24"
  availability_zone = "ap-northeast-2d"

  tags = {
    Name = "private-ca-central-1b"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Musafar"
  }
}

resource "aws_subnet" "private-ap-northeast-2c" {
  vpc_id            = aws_vpc.app1.id
  cidr_block        = "10.74.13.0/24"
  availability_zone = "ap-northeast-2c"

  tags = {
    Name = "private-ca-central-1d"
    Service = "application1"
    Owner = "Chewbacca"
    Planet = "Musafar"
  }
}

```

## Step 4: Internet Gateway

Establish an Internet Gateway (IG) to allow communication between your VPC and the internet.

``` hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.app1.id

  tags = {
    Name    = "app1_IG"
    Service = "application1"
    Owner   = "Luke"
    Planet  = "Musafar"
  }
}

```

## Step 5: NAT Gateway

Implement a NAT Gateway to enable instances in a private subnet to connect to the internet or other AWS services but prevent the internet from initiating connections with the instances.

```hcl
resource "aws_eip" "nat" {
  vpc = true

  tags = {
    Name = "nat"
  }
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public-ap-northeast-2a.id

  tags = {
    Name = "nat"
  }

  depends_on = [aws_internet_gateway.igw]
}
```

## Step 6: Route Tables

Configure route tables to define rules for traffic routing within your VPC.

```hcl
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.app1.id

  route = [
    {
      cidr_block                 = "0.0.0.0/0"
      nat_gateway_id             = aws_nat_gateway.nat.id
      carrier_gateway_id         = ""
      destination_prefix_list_id = ""
      egress_only_gateway_id     = ""
      gateway_id                 = ""
      instance_id                = ""
      ipv6_cidr_block            = ""
      local_gateway_id           = ""
      network_interface_id       = ""
      transit_gateway_id         = ""
      vpc_endpoint_id            = ""
      vpc_peering_connection_id  = ""
    },
  ]

  tags = {
    Name = "private"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.app1.id

  route = [
    {
      cidr_block                 = "0.0.0.0/0"
      gateway_id                 = aws_internet_gateway.igw.id
      nat_gateway_id             = ""
      carrier_gateway_id         = ""
      destination_prefix_list_id = ""
      egress_only_gateway_id     = ""
      instance_id                = ""
      ipv6_cidr_block            = ""
      local_gateway_id           = ""
      network_interface_id       = ""
      transit_gateway_id         = ""
      vpc_endpoint_id            = ""
      vpc_peering_connection_id  = ""
    },
  ]

  tags = {
    Name = "public"
  }
}

resource "aws_route_table_association" "private-ap-northeast-2a" {
  subnet_id      = aws_subnet.private-ap-northeast-2a.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private-ap-northeast-2d" {
  subnet_id      = aws_subnet.private-ap-northeast-2d.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "public-ap-northeast-2a" {
  subnet_id      = aws_subnet.public-ap-northeast-2a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public-ap-northeast-2d" {
  subnet_id      = aws_subnet.public-ap-northeast-2d.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public-ap-northeast-2c" {
  subnet_id      = aws_subnet.public-ap-northeast-2c.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private-ap-northeast-2c" {
  subnet_id      = aws_subnet.private-ap-northeast-2c.id
  route_table_id = aws_route_table.private.id
}

```

## Step 7: Security Groups

Define security groups to control the traffic to and from your instances. You'll want to allow web traffic and restrict all unnecessary access.

```hcl
resource "aws_security_group" "app1-sg01-servers" {
  name        = "app1-sg01-servers"
  description = "app1-sg01-servers"
  vpc_id      = aws_vpc.app1.id

  ingress {
    description = "MyHomePage"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "SecureMyHomePage"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "MyEvilBox"
    from_port   = 3389
    to_port     = 3389
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
    Name    = "app1-sg01-servers"
    Service = "application1"
    Owner   = "Luke"
    Planet  = "Musafar"
  }

}





resource "aws_security_group" "app1-sg02-LB01" {
  name        = "app1-sg02-LB01"
  description = "app1-sg02-LB01"
  vpc_id      = aws_vpc.app1.id

  ingress {
    description = "MyHomePage"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SecureMyHomePage"
    from_port   = 443
    to_port     = 443
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
    Name    = "app1-sg02-LB01"
    Service = "application1"
    Owner   = "Luke"
    Planet  = "Musafar"
  }

}
```

## Step 8: Launch Template

Create a launch template for your EC2 instances. This will define the AMI, instance type, and the security group among other settings. Also when adding user data you have to encode it using base64. To do that use the following command to encode it and to output it to a text file. replace path to file with the relative path to your user data file. 

```bash
base64 /path/to/file > output.txt

```

```hcl 
resource "aws_launch_template" "app1_LT" {
  name_prefix   = "app1_LT"
  image_id      = "ami-0e01e66dacaf1454d"  
  instance_type = "t2.micro"

  key_name = "MyLinuxBox"

  vpc_security_group_ids = [aws_security_group.app1-sg01-servers.id]

  user_data = "${file("output.txt")}"

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name    = "app1_LT"
      Service = "application1"
      Owner   = "Chewbacca"
      Planet  = "Mustafar"
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}

```

## Step 9: Target Group

Set up a target group for your load balancer to direct traffic to. The target group defines health check settings and which instances should receive traffic.

``` hcl
resource "aws_lb_target_group" "app1_tg" {
  name     = "app1-target-group"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.app1.id
  target_type = "instance"

  health_check {
    enabled             = true
    interval            = 30
    path                = "/"
    protocol            = "HTTP"
    healthy_threshold   = 10
    unhealthy_threshold = 2
    timeout             = 5
    matcher             = "200"
  }

  tags = {
    Name    = "App1TargetGroup"
    Service = "App1"
    Owner   = "User"
    Project = "Web Service"
  }
}

```

## Step 10: Load Balancer

Deploy an Application Load Balancer (ALB) to distribute incoming application traffic across multiple targets, such as EC2 instances.

```hcl
resource "aws_lb" "app1_alb" {
  name               = "app1-load-balancer"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.app1-sg02-LB01.id]
  subnets            = [
    aws_subnet.public-ap-northeast-2a.id,
    aws_subnet.public-ap-northeast-2c.id,
    aws_subnet.public-ap-northeast-2d.id,
  ]
  enable_deletion_protection = false

  tags = {
    Name    = "App1LoadBalancer"
    Service = "App1"
    Owner   = "User"
    Project = "Web Service"
  }
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.app1_alb.arn
  port              = 80
  protocol          = "HTTP"

  # default_action {
  #   type             = "forward"
  #   target_group_arn = aws_lb_target_group.app1_tg.arn
  # }

    default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}
```

## Step 11: Auto Scaling Group

Create an Auto Scaling Group (ASG) that uses the launch template and target group to maintain application availability and scale your EC2 instances automatically.

```hcl
resource "aws_autoscaling_group" "app1_asg" {
  name_prefix           = "app1-auto-scaling-group-"
  min_size              = 3
  max_size              = 15
  desired_capacity      = 6
  vpc_zone_identifier   = [
    aws_subnet.private-ap-northeast-2a.id,
    aws_subnet.private-ap-northeast-2c.id,
    aws_subnet.private-ap-northeast-2d.id,
  ]
  health_check_type          = "ELB"
  health_check_grace_period  = 300
  force_delete               = true
  target_group_arns          = [aws_lb_target_group.app1_tg.arn]

  launch_template {
    id      = aws_launch_template.app1_LT.id
    version = "$Latest"
  }

  enabled_metrics = ["GroupMinSize", "GroupMaxSize", "GroupDesiredCapacity", "GroupInServiceInstances", "GroupTotalInstances"]

  # Instance protection for launching
  initial_lifecycle_hook {
    name                  = "instance-protection-launch"
    lifecycle_transition  = "autoscaling:EC2_INSTANCE_LAUNCHING"
    default_result        = "CONTINUE"
    heartbeat_timeout     = 60
    notification_metadata = "{\"key\":\"value\"}"
  }

  # Instance protection for terminating
  initial_lifecycle_hook {
    name                  = "scale-in-protection"
    lifecycle_transition  = "autoscaling:EC2_INSTANCE_TERMINATING"
    default_result        = "CONTINUE"
    heartbeat_timeout     = 300
  }

  tag {
    key                 = "Name"
    value               = "app1-instance"
    propagate_at_launch = true
  }

  tag {
    key                 = "Environment"
    value               = "Production"
    propagate_at_launch = true
  }
}


# Auto Scaling Policy
resource "aws_autoscaling_policy" "app1_scaling_policy" {
  name                   = "app1-cpu-target"
  autoscaling_group_name = aws_autoscaling_group.app1_asg.name

  policy_type = "TargetTrackingScaling"
  estimated_instance_warmup = 120

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 75.0
  }
}

# Enabling instance scale-in protection
resource "aws_autoscaling_attachment" "app1_asg_attachment" {
  autoscaling_group_name = aws_autoscaling_group.app1_asg.name
  alb_target_group_arn   = aws_lb_target_group.app1_tg.arn
}

```


## Step 12: Route 53

Utilize AWS Route 53 to create a DNS record that points to your ALB, providing a friendly domain name for your users.

```hcl
data "aws_route53_zone" "public" {
  name         = "abudevops.com"
  private_zone = false
}

resource "aws_acm_certificate" "api" {
  domain_name       = "abudevops.com"
  validation_method = "DNS"
}

resource "aws_route53_record" "api_validation" {
  for_each = {
    for dvo in aws_acm_certificate.api.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.public.zone_id
}

resource "aws_acm_certificate_validation" "api" {
  certificate_arn         = aws_acm_certificate.api.arn
  validation_record_fqdns = [for record in aws_route53_record.api_validation : record.fqdn]
}

resource "aws_route53_record" "api" {
  name    = aws_acm_certificate.api.domain_name
  type    = "A"
  zone_id = data.aws_route53_zone.public.zone_id

  alias {
    name                   = aws_lb.app1_alb.dns_name
    zone_id                = aws_lb.app1_alb.zone_id
    evaluate_target_health = false
  }
}

resource "aws_lb_listener" "my_app_eg2_tls" {
  load_balancer_arn = aws_lb.app1_alb.arn
  port              = "443"
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate.api.arn
  ssl_policy        = "ELBSecurityPolicy-2016-08"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app1_tg.arn
  }

  depends_on = [aws_acm_certificate_validation.api]
}

output "custom_domain" {
  value = "https://${aws_acm_certificate.api.domain_name}"
}
```

## Step 13: Web Application Firewall (WAF)

Lastly, enhance security by implementing a Web Application Firewall (WAF). This will protect your web application from common web exploits.

```hcl
resource "aws_wafv2_web_acl" "WafWebAcl" {
  name  = "wafv2-web-acl"
  scope = "REGIONAL"

  default_action {
    allow {}
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "WAF_Common_Protections"
    sampled_requests_enabled   = true
  }

  rule {
    name     = "AWS-AWSManagedRulesCommonRuleSet"
    priority = 0
    override_action {
      none {}
    }
    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
        # Excluded rules can be specified here if needed
        # excluded_rules {
        #   name = "SizeRestrictions_BODY"
        # }
        # excluded_rules {
        #   name = "NoUserAgent_HEADER"
        # }
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesCommonRuleSet"
      sampled_requests_enabled   = true
    }
  }

  # ... other rules here ...

  # Example for additional rules (make sure to increment the priority for each rule)
  rule {
    name     = "AWS-AWSManagedRulesLinuxRuleSet"
    priority = 1
    override_action {
      none {}
    }
    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesLinuxRuleSet"
        vendor_name = "AWS"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesLinuxRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesWindowsRuleSet"
    priority = 10
    override_action {
      none {}
    }
    statement {
      managed_rule_group_statement {
        name        = "AWS-AWSManagedRulesWindowsRuleSet"
        vendor_name = "AWS"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesWindowsRuleSet"
      sampled_requests_enabled   = true
    }
  }

  rule {
    name     = "AWS-AWSManagedRulesKnownBadInputsRuleSet"
    priority = 20
    override_action {
      none {}
    }
    statement {
      managed_rule_group_statement {
        name        = "AWS-AWSManagedRulesKnownBadInputsRuleSet"
        vendor_name = "AWS"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "AWS-AWSManagedRulesKnownBadInputsRuleSet"
      sampled_requests_enabled   = true
    }
  }  

  # Continue to define other rules as needed, incrementing the priority each time

  # Don't forget to close the resource block with a closing brace.
}

resource "aws_cloudwatch_log_group" "WafWebAclLoggroup" {
  name              = "aws-waf-logs-wafv2-web-acl"
  retention_in_days = 30
}

resource "aws_wafv2_web_acl_logging_configuration" "WafWebAclLogging" {
  log_destination_configs = [aws_cloudwatch_log_group.WafWebAclLoggroup.arn]
  resource_arn            = aws_wafv2_web_acl.WafWebAcl.arn
}

resource "aws_wafv2_web_acl_association" "WafWebAclAssociation" {
  resource_arn = aws_lb.app1_alb.arn  # Ensure that this resource ARN is defined elsewhere in your Terraform
  web_acl_arn  = aws_wafv2_web_acl.WafWebAcl.arn
}

```
???+ info "How to run Terraform"
    To run and destroy terraform, use the following commands:

    1. terraform init

    2. terraform plan

    3. terraform apply

    4. terraform destroy

![2023-11-08 23-25-20.gif](https://raw.githubusercontent.com/AMohamed0/CloudDocs/main/screenshots/2023-11-08%2023-25-20.gif)

## Conclusion

As we conclude this tutorial, you now have the knowledge to create a highly scalable and secure website on AWS using Terraform. You've learned how to set up the necessary infrastructure components step by step, from provisioning a VPC to deploying an Auto Scaling Group and securing your environment with WAF and Route 53.

![Congratulations Image](https://www.icegif.com/wp-content/uploads/2023/05/icegif-1098.gif)
