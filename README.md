# Story 5 — PostgreSQL Memory Alert Integration

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0089D6?style=for-the-badge&logo=microsoftazure&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Status](https://img.shields.io/badge/Status-In_Progress-yellow?style=for-the-badge)

## Ticket Reference
| Field | Value |
|---|---|
| **Story** | Story 5 |
| **Title** | Update PostgreSQL Flexible Server provisioning script to create memory alerts |
| **Sprint** | EDAV PostgreSQL Alert Automation |
| **Assignee** | Austin Jones |
| **Reviewer** | Krishna |

---

## Purpose

Integrate memory usage alerts into the PostgreSQL Flexible Server provisioning workflow using the `global_metric_alert` module (Story 2). When a new PostgreSQL Flexible Server is provisioned, memory alerts at both **critical (>85%)** and **warning (>70%)** thresholds are automatically created.

---

## Alert Definitions

| Alert Name | Metric | Threshold | Severity | Frequency |
|---|---|---|---|---|
| `postgres-memory-critical-{env}` | `memory_percent` | > 85% | 1 (Error) | PT1M |
| `postgres-memory-warning-{env}` | `memory_percent` | > 70% | 2 (Warning) | PT1M |

---

## Terraform Code

```hcl
# Memory Critical Alert (> 85%)
module "postgres_memory_critical" {
  source = "../../modules/global_metric_alert"

  alert_name          = "postgres-memory-critical-${var.environment}"
  resource_group_name = var.resource_group_name
  description         = "PostgreSQL memory usage exceeded 85% — OOM risk, requires immediate review"

  scopes = [var.postgres_server_resource_id]

  severity         = 1
  frequency        = "PT1M"
  window_size      = "PT5M"
  metric_namespace = "Microsoft.DBforPostgreSQL/flexibleServers"
  metric_name      = "memory_percent"
  aggregation      = "Average"
  operator         = "GreaterThan"
  threshold        = 85
  action_group_id  = var.action_group_id

  tags = {
    Environment = var.environment
    Story       = "story-5-memory-alert-integration"
    ManagedBy   = "Terraform"
    Component   = "PostgreSQL"
    AlertType   = "Memory-Critical"
  }
}

# Memory Warning Alert (> 70%)
module "postgres_memory_warning" {
  source = "../../modules/global_metric_alert"

  alert_name          = "postgres-memory-warning-${var.environment}"
  resource_group_name = var.resource_group_name
  description         = "PostgreSQL memory usage exceeded 70% — review connection pool and query load"

  scopes = [var.postgres_server_resource_id]

  severity         = 2
  frequency        = "PT1M"
  window_size      = "PT5M"
  metric_namespace = "Microsoft.DBforPostgreSQL/flexibleServers"
  metric_name      = "memory_percent"
  aggregation      = "Average"
  operator         = "GreaterThan"
  threshold        = 70
  action_group_id  = var.action_group_id

  tags = {
    Environment = var.environment
    Story       = "story-5-memory-alert-integration"
    ManagedBy   = "Terraform"
    Component   = "PostgreSQL"
    AlertType   = "Memory-Warning"
  }
}
```

---

## How to Run Locally

### Prerequisites
- [Terraform >= 1.5](https://developer.hashicorp.com/terraform/downloads)
- - Azure CLI authenticated (`az login`)
  - - `global_metric_alert` module available (Story 2)
    - - PostgreSQL Flexible Server provisioned in DEV
     
      - ### Steps
     
      - ```bash
        # 1. Clone the repo
        git clone https://github.com/ausjones84/story-5-postgres-memory-alert-integration.git
        cd story-5-postgres-memory-alert-integration

        # 2. Navigate to DEV environment
        cd terraform-scripts/dev/postgres-alerts

        # 3. Copy and edit tfvars
        cp postgres-servers.tfvars.example postgres-servers.tfvars
        # Edit: postgres_server_resource_id, resource_group_name, action_group_id, environment

        # 4. Init, validate, plan, apply
        terraform init -backend=false
        terraform validate
        terraform plan -var-file="postgres-servers.tfvars"
        terraform apply -var-file="postgres-servers.tfvars"
        ```

        ### Expected Output
        ```
        azurerm_monitor_metric_alert.postgres_memory_critical: Creation complete
        azurerm_monitor_metric_alert.postgres_memory_warning: Creation complete

        Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
        ```

        ---

        ## Acceptance Criteria Checklist

        - [ ] Memory alert module calls added to DEV environment script
        - [ ] - [ ] `memory_percent` metric used with `GreaterThan` operator
        - [ ] - [ ] Critical threshold = **85%** with severity **1**
        - [ ] - [ ] Warning threshold = **70%** with severity **2**
        - [ ] - [ ] Action group configured per environment
        - [ ] - [ ] `terraform plan` shows 2 new alert resources
        - [ ] - [ ] Alerts validated in Azure Portal after apply
        - [ ] - [ ] Ticket updated
       
        - [ ] ---
       
        - [ ] ## Ticket Update (Copy-Paste Ready)
       
        - [ ] ```
        - [ ] Integrated PostgreSQL memory alert definitions using the global metric alert module.
        - [ ] Added memory usage alert at >85% (severity 1/Error) and >70% (severity 2/Warning) using
        - [ ] the memory_percent metric from Microsoft.DBforPostgreSQL/flexibleServers namespace.
       
        - [ ] Both alerts use 1-minute evaluation frequency with 5-minute aggregation window.
        - [ ] Validated with terraform plan — 2 new resources will be created.
        - [ ] ```
       
        - [ ] ---
       
        - [ ] ## Related Stories
       
        - [ ] | Story | Repo | Description |
        - [ ] |---|---|---|
        - [ ] | Story 1 | [story-1-postgres-alert-poc-evaluation](https://github.com/ausjones84/story-1-postgres-alert-poc-evaluation) | POC evaluation |
        - [ ] | Story 2 | [story-2-global-metric-alert-module](https://github.com/ausjones84/story-2-global-metric-alert-module) | Global metric alert module (dependency) |
        - [ ] | Story 4 | [story-4-postgres-cpu-alert-integration](https://github.com/ausjones84/story-4-postgres-cpu-alert-integration) | CPU alert integration |
        - [ ] | Story 5 | **This repo** | Memory alert integration |
        - [ ] | Story 6 | [story-6-postgres-disk-alert-integration](https://github.com/ausjones84/story-6-postgres-disk-alert-integration) | Disk alert integration |
        - [ ] | Story 7 | [story-7-postgres-connection-alert-integration](https://github.com/ausjones84/story-7-postgres-connection-alert-integration) | Connection alert integration |
        - [ ] | Story 8 | [story-8-postgres-restore-alert-rules](https://github.com/ausjones84/story-8-postgres-restore-alert-rules) | Restore alert rules |
