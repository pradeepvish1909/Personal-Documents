resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = var.environment
  }
}
resource "aws_subnet" "this" {
  for_each = var.subnets

  vpc_id            = aws_vpc.this.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az
}
resource "aws_db_subnet_group" "sub_grp" {
  name       = "${var.environment}-db-subnet-group"
  subnet_ids = [for s in aws_subnet.this : s.id]

  tags = {
    Name = "My DB subnet group"
  }
}
resource "aws_db_instance" "default" {
  allocated_storage            = var.db_allocated_storage
  db_name                      = var.db_name
  identifier                   = var.db_identifier
  engine                       = var.db_engine
  engine_version               = var.db_engine_version
  instance_class               = var.db_instance_class
  username                     = "test"
  manage_master_user_password  = true

  db_subnet_group_name = aws_db_subnet_group.sub_grp.name
  #parameter_group_name = var.parameter_group_name

  backup_retention_period = var.backup_retention_period
  backup_window           = var.backup_window

  #monitoring_interval = var.monitoring_interval
  #monitoring_role_arn = aws_iam_role.rds_monitoring.arn

  maintenance_window  = var.maintenance_window
  deletion_protection = var.deletion_protection
  skip_final_snapshot = var.skip_final_snapshot
}

resource "aws_s3_bucket" "name" {
    bucket = var.bucket
  
}
