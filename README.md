# ---------- General ----------
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

# ---------- VPC ----------
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "subnets" {
  description = "Subnet configuration"
  type = map(object({
    cidr = string
    az   = string
  }))
}

# ---------- RDS ----------
variable "db_identifier" {
  description = "RDS instance identifier"
  type        = string
  default = "devtest"
}

variable "db_name" {
  description = "Initial database name"
  type        = string
}

variable "db_engine" {
  description = "Database engine"
  type        = string
  default     = "mysql"
}

variable "db_engine_version" {
  description = "Database engine version"
  type        = string
  default     = "8.0"
}

variable "db_instance_class" {
  description = "RDS instance class"
  type        = string
  default = ""
}

variable "db_allocated_storage" {
  description = "Allocated storage in GB"
  type        = number
  default = 8
}

variable "db_username" {
  description = "Master DB username"
  type        = string
  default = ""
}

# variable "parameter_group_name" {
#   description = "DB parameter group"
#   type        = string
# }

variable "backup_retention_period" {
  description = "Backup retention in days"
  type        = number
  default     = 7
}

variable "backup_window" {
  description = "Preferred backup window"
  type        = string
  default = ""
}

variable "maintenance_window" {
  description = "Maintenance window"
  type        = string
  default = ""
}

# variable "monitoring_interval" {
#   description = "Enhanced monitoring interval"
#   type        = number
#   default     = 60
# }

variable "deletion_protection" {
  description = "Enable deletion protection"
  type        = bool
  default     = true
}

variable "skip_final_snapshot" {
  description = "Skip final snapshot on delete"
  type        = bool
  default     = true
}

variable "bucket" {
  default = ""
  type = string
}
