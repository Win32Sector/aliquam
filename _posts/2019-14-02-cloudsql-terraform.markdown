---
layout: post
title: Поднимаем CloudSQL instance в Google cloud с помощью Terraform
category: article
comments: true
description: Поднимаем CloudSQL instance в Google cloud с помощью Terraform
tags:
    - google cloud
    - terraform
    - cloudsql
---

## Terraform файлы для поднятия инстанса CloudSQL

#### main.tf
```
    resource "google_sql_database_instance" "sql_database_instance_mysql" {
        name                             = "${lower(var.name)}-${lower(var.environment)}-${count.index+1}"
        project                          = "${var.project}"
        database_version                 = "${var.database_version}"
        region                           = "${lookup(var.region, var.database_version)}"

        master_instance_name             = "${var.master_instance_name}"

        settings {
            tier                        = "${lookup(var.settings_tier, var.database_version)}"
            activation_policy           = "${var.settings_activation_policy}"
            authorized_gae_applications = ["${var.settings_authorized_gae_applications}"]
            availability_type           = "${var.settings_availability_type}"
            crash_safe_replication      = "${var.settings_crash_safe_replication}"
            replication_type            = "${var.settings_replication_type}"
            pricing_plan                = "${var.settings_pricing_plan}"


            database_flags  = ["${var.settings_database_flags}"]

            backup_configuration {
                binary_log_enabled  = "${var.settings_backup_configuration_binary_log_enabled}"
                enabled             = "${var.settings_backup_configuration_enabled}"
                start_time          = "${var.settings_backup_configuration_start_time}"
            }

            ip_configuration {
                ipv4_enabled    = "${var.settings_ip_configuration_ipv4_enabled}"
                require_ssl     = "${var.settings_ip_configuration_require_ssl}"

                authorized_networks {
                    name            = "${var.settings_authorized_networks_name}"
                    value           = "${var.settings_authorized_networks_value}"
                }
            }

            location_preference {
                follow_gae_application  = "${var.settings_location_preference_follow_gae_application}"
                zone                    = "${var.settings_location_preference_zone}"
            }

            maintenance_window = []

        }
    }

```

#### database.tf

```
    resource "google_sql_database" "sql_database" {
        name        = "${var.sql_database_name}"
        project     = "${var.project}"
        instance    = "${var.sql_database_instance_name}"
        charset     = "${var.sql_database_charset}"
        collation   = "${var.sql_database_collation}"
    }
```

#### variables.tf

```
    variable "name" {
        description = "A unique name for the resource, required by GCE. Changing this forces a new resource to be created."
        default     = "test"
    }

    variable "environment" {
        description = "Environment for service"
        default     = "staging"
    }

    variable "database_version" {
        description = "(Optional, Default: MYSQL_5_6) The MySQL version to use. Can be MYSQL_5_6, MYSQL_5_7 or POSTGRES_9_6 for second-generation instances, or MYSQL_5_5 or MYSQL_5_6 for first-generation instances. See Second Generation Capabilities for more information. POSTGRES_9_6 support is in Beta."
        default     = "MYSQL_5_6"
    }

    variable "region" {
        description = "(Required)"
        type        = "map"
        default     = {
            MYSQL_5_6       = "europe-west1"
        }
    }

    variable "project" {
        description = "(Optional) The ID of the project in which the resource belongs. If it is not provided, the provider project is used."
        default     = "DEFAULT"
    }


    variable "settings_tier" {
        description = "(Required) The machine tier (First Generation) or type (Second Generation) to use. See tiers for more details and supported versions. Postgres supports only shared-core machine types such as db-f1-micro, and custom machine types such as db-custom-2-13312. See the Custom Machine Type Documentation to learn about specifying custom machine types."
        type        = "map"
        default     = {
            MYSQL_5_6       = "db-n1-standard-2"
        }
    }

    variable "settings_activation_policy" {
        description = "(Optional) This specifies when the instance should be active. Can be either ALWAYS, NEVER or ON_DEMAND."
        default     = "ALWAYS"
    }

    variable "settings_availability_type" {
        description = "(Optional) This specifies whether a PostgreSQL instance should be set up for high availability (REGIONAL) or single zone (ZONAL)."
        default     = "ZONAL"
    }

    variable "settings_crash_safe_replication" {
        description = "(Optional) Specific to read instances, indicates when crash-safe replication flags are enabled."
        default     = ""
    }

    variable "settings_disk_autoresize" {
        description = "(Optional, Second Generation, Default: true) Configuration to increase storage size automatically."
        default     = "false"
    }

    variable "settings_disk_size" {
        description = "(Optional, Second Generation, Default: 10) The size of data disk, in GB. Size of a running instance cannot be reduced but can be increased."
        default     = "20"
    }

    variable "settings_disk_type" {
        description = "(Optional, Second Generation, Default: PD_SSD) The type of data disk: PD_SSD or PD_HDD."
        default     = "PD_SSD"
    }

    variable "settings_pricing_plan" {
        description = "(Optional, First Generation) Pricing plan for this instance, can be one of PER_USE or PACKAGE."
        default     = "PER_USE"
    }

    variable "settings_replication_type" {
        description = "(Optional) Replication type for this instance, can be one of ASYNCHRONOUS or SYNCHRONOUS."
        default     = "SYNCHRONOUS"
    }

    variable "settings_backup_configuration_binary_log_enabled" {
        description = "(Optional) True if binary logging is enabled. If logging is false, this must be as well."
        default     = "false"
    }

    variable "settings_backup_configuration_enabled" {
        description = "(Optional) True if backup configuration is enabled."
        default     = "true"
    }

    variable "settings_backup_configuration_start_time" {
        description = "(Optional) HH:MM format time indicating when backup configuration starts."
        default     = "23:00"
    }

    variable "settings_ip_configuration_ipv4_enabled" {
        description = "(Optional) True if the instance should be assigned an IP address. The IPv4 address cannot be disabled for Second Generation instances."
        default     = "true"
    }

    variable "settings_ip_configuration_require_ssl" {
        description = "(Optional) True if mysqld should default to REQUIRE X509 for users connecting over IP."
        default     = ""
    }


    variable "settings_authorized_networks_value" {
        description = "(Optional) A CIDR notation IPv4 or IPv6 address that is allowed to access this instance. Must be set even if other two attributes are not for the whitelist to become active."
        default     = ""
    }

    variable "settings_location_preference_follow_gae_application" {
        description = "(Optional) A GAE application whose zone to remain in. Must be in the same region as this instance."
        default     = ""
    }

    variable "settings_location_preference_zone" {
        description = "(Optional) The preferred compute engine zone."
        default     = ""
    }

    variable "settings_database_flags" {
        description = "Set database flags"
        default     = []
    }

    variable "enable_replication" {
        description = "Enable replication"
        default     = "false"
    }

    variable "enable_sql_database_creating" {
        description = "Enable sql DB creating"
        default     = "true"
    }

    variable "sql_database_name" {
        description = "(Required) The name of the database."
        default     = "test_db"
    }

    variable "sql_database_instance_name" {
        description = "(Required) The name of containing instance."
        default     = ""
    }

    variable "sql_database_charset" {
        description = "(Optional) The charset value. See MySQL's Supported Character Sets and Collations and Postgres' Character Set Support for more details and supported values. Postgres databases are in Beta, and have limited charset support; they only support a value of UTF8 at creation time."
        default     = ""
    }

    variable "sql_database_collation" {
        description = "(Optional) The collation value. See MySQL's Supported Character Sets and Collations and Postgres' Collation Support for more details and supported values. Postgres databases are in Beta, and have limited collation support; they only support a value of en_US.UTF8 at creation time."
        default     = ""
    }

    variable "enable_sql_user_creating" {
        description = "Enable sql user"
        default     = "true"
    }

    variable "sql_user_name" {
        description = "(Required) The name of the user. Changing this forces a new resource to be created."
        default     = "test_user"
    }

    variable "sql_user_host" {
        description = "(Optional) The host the user can connect from. This is only supported for MySQL instances. Don't set this field for PostgreSQL instances. Can be an IP address. Changing this forces a new resource to be created."
        default     = ""
    }

    variable "sql_user_password" {
        description = "(Optional) The password for the user. Can be updated."
        default     = ""
    }

```