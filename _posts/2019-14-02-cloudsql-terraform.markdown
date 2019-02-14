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