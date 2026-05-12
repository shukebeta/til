# Cloning an EC2 Instance: Three Ways

AWS console offers three ways to duplicate an EC2 instance, differing in whether disk data is carried over.

**Create AMI (recommended, full clone)** — preserves the system disk, installed software, and all configuration.

In the EC2 console, select the target instance → Actions → Image and templates → Create image. Wait for the AMI status to become `available` (a few minutes to tens of minutes), then go to AMIs → select it → Launch instance from AMI. Adjust instance type, subnet, Security Group as needed.

**Launch More Like This (fastest, no data)** — copies only instance configuration (type, SG, subnet, tags). The system disk is brand new.

Actions → Image and templates → Launch more like this. The Launch page opens with config pre-filled; confirm and launch. Good for stateless instances, e.g. web servers initialized via userdata.

**Launch Template** — if the original instance had a Launch Template saved, launch directly from it. EC2 → Launch Templates → select template → Actions → Launch instance from template.

Use AMI for most cases. Use Launch More Like This when you only need the same specs with a clean disk.
