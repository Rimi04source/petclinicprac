# Variables

You can find below all variables that are part of this module.

## Required
These inputs are mandatory for a successful deployment (enforced by module preconditions). Defaults shown for some are placeholders and must be overridden for your environment.

| Name | Type | Values | Description |
|--|--|--|--|
| `aks_core` | `object` |  | Core AKS settings (naming, placement, DNS prefix, SKU tier, managed node resource group). Provide environment-specific values including a valid `subnet_id`. |
| `disk_encryption` | `object` |  | Customer-managed key (CMK) configuration. Must set either `existing_des_id` or `create.key_vault_key_id`. Key ID format: `/subscriptions/{SUB}/resourceGroups/{RG}/providers/Microsoft.KeyVault/vaults/{KV}/keys/{KEY}/{VERSION}`. |
| `gso_logs_storage_account_id` | `string` | Example: `/subscriptions/{SUB}/resourceGroups/{RG}/providers/Microsoft.Storage/storageAccounts/{NAME}` | Storage account ARM ID used for control-plane diagnostics streaming. Must be non-empty. |

## Optional

These variables are available for user to change in order to reach the desired configuration of this module. If not provided by the user, these will adopt their default value.

| Name | Type | Values | Default | Description |
|--|--|--|--|--|
| `resource_prefix` | `string` | Examples: `gso`, `aks` | `null` | Optional prefix prepended to the names of created resources. |
| `kubernetes_version` | `string` | Examples: `1.29.7`, `1.28.11` | `null` | Control plane version; `null` lets Azure select the latest allowed version. |
| `support_plan` | `string` | `KubernetesOfficial`, `AKSLongTermSupport` | `"KubernetesOfficial"` | AKS support plan. |
| `tags` | `map(string)` | Example: `{ environment = "prod", owner = "platform" }` | `{}` | Common tags merged into all managed resources. |
| `aks_identity` | `object` |  | `{ type = "SystemAssigned", user_assigned = { id = "", name = "aks-identity" }, workload_identity_enabled = true, oidc_issuer_enabled = true }` | Control plane managed identity configuration and Workload Identity (OIDC) toggles. `type` must be `SystemAssigned` or `UserAssigned` (Service Principal not allowed). |
| `aad_rbac` | `object` |  | `{ admin_group_object_ids = [], azure_rbac_enabled = false, tenant_id = "" }` | Azure AD admin groups and whether to enable Azure RBAC for Kubernetes. |
| `storage_profile` | `object` |  | `{ blob_driver_enabled = true, disk_driver_enabled = true, file_driver_enabled = true, snapshot_controller_enabled = true }` | Controls AKS storage drivers. |
| `os_profiles` | `object` |  | `{ linux = { enabled = false, admin_username = "azureuser", ssh_keys = [] }, windows = { enabled = false, admin_username = "azureuser", admin_password = "" } }` | Linux/Windows local admin credentials. |
| `aks_availability` | `object` |  | `{ enabled = true, cluster_zones = ["1","2","3"], node_pool_distribution = "all" }` | Availability zones. `node_pool_distribution` supports `single`, `multiple`, or `all`. |
| `aks_network` | `object` |  | `{ plugin = "azure", policy = "azure", load_balancer_sku = "Standard", service_cidr = "10.0.0.0/16", dns_service_ip = "10.0.0.10", pod_cidr = "10.244.0.0/16", outbound_type = "userDefinedRouting", network_mode = "", network_data_plane = "", network_plugin_mode = "" }` | Cluster network profile. `plugin` in `azure`/`kubenet`; `policy` in `azure`/`calico`; `outbound_type` in `loadBalancer`/`userDefinedRouting`. |
| `node_pools` | `map(object)` |  | `{ system = { name = "systempool", mode = "System", vm_size = "Standard_D4s_v3", enable_auto_scaling = false, node_count = 2, min_count = 2, max_count = 3, os_disk_size_gb = 128, os_disk_type = "Managed", availability_zones = ["1"], enable_fips = false, os_sku = "Ubuntu", os_type = "Linux", node_labels = {}, tags = {}, priority = "Regular", eviction_policy = "Delete", spot_max_price = -1, max_pods = 30, node_taints = [], pod_subnet_id = "", proximity_placement_group_id = "", ultra_ssd_enabled = false, windows_profile = { outbound_nat_enabled = true }, gpu_instance_type = "" } }` | Node pool definitions (must include key `system`). For nested fields, refer to AKS provider docs for valid values (e.g., `mode` `System`/`User`, `os_type` `Linux`/`Windows`, `priority` `Regular`/`Spot`). |
| `upgrade` | `object` |  | `{ channel = "stable", node_os_auto_upgrade = true, temporary_name_for_rotation = null, surge = { system = "", user = "" }, maintenance = { enabled = true, window = { allowed_days = ["Saturday","Sunday"], allowed_hours = [2,3,4,5], not_allowed = [] } }, maintenance_auto_upgrade = { enabled = false, window = { frequency = "Weekly", interval = 1, duration = 4, day_of_week = "Sunday", start_time = "02:00", utc_offset = "+00:00", not_allowed = [] } }, maintenance_node_os = { enabled = false, window = { frequency = "Weekly", interval = 1, duration = 4, day_of_week = "Saturday", start_time = "02:00", utc_offset = "+00:00", not_allowed = [] } } }` | Upgrade channel and maintenance windows. `channel` supports `patch`, `stable`, `rapid`. |
| `autoscaler` | `object` |  | `{ enabled = false, balance_similar_node_groups = false, daemonset_eviction_for_empty_nodes_enabled = false, daemonset_eviction_for_occupied_nodes_enabled = true, expander = "random", ignore_daemonsets_utilization_enabled = false, max_graceful_termination_sec = "600", max_node_provisioning_time = "15m", max_unready_nodes = "3", max_unready_percentage = "45", new_pod_scale_up_delay = "10s", scale_down_delay_after_add = "10m", scale_down_delay_after_delete = "10s", scale_down_delay_after_failure = "3m", scan_interval = "10s", scale_down_unneeded = "10m", scale_down_unready = "20m", scale_down_utilization_threshold = "0.5", empty_bulk_delete_max = "10", skip_nodes_with_local_storage = true, skip_nodes_with_system_pods = true }` | Cluster autoscaler profile (durations are strings like `10m`). |
| `workload` | `object` |  | `{ keda_enabled = true, vpa_enabled = true, key_vault_csi = { enable_secret_rotation = true, rotation_interval = "2m" }, image_cleaner = { enabled = true, interval_hours = 24 }, cost_analysis_enabled = true }` | Workload add-ons (KEDA, VPA, Key Vault CSI, image cleaner, cost analysis). |
| `backup` | `object` |  | `{ enabled = false, vault_id = "", policy_id = "" }` | Azure Backup for AKS. `vault_id` format: `/subscriptions/{SUB}/resourceGroups/{RG}/providers/Microsoft.DataProtection/backupVaults/{VAULT}`. |
| `integrations` | `object` |  | `{ acr = { ids = [] } }` | Optional integrations (e.g., ACR IDs to grant `AcrPull`). |
| `lb_validation` | `object` |  | `{ id = "", name = "", resource_group_name = "" }` | Governance-only: validates an existing Load Balancer is internal-only. |
| `nrg_lockdown` | `object` |  | `{ enabled = true, restriction_level = "ReadOnly" }` | Apply a management lock on the AKS-managed node resource group. |

## Enforced

These defaults back GSO mandates via resource settings and/or preconditions. They are not required inputs, but altering them typically needs approval. Review before changing.

| Name | Type | Values | Default | Description |
|--|--|--|--|--|
| `aks_security` | `object` |  | `{ private_cluster_enabled = true, private_cluster_public_fqdn_enabled = false, local_account_disabled = true, host_encryption_enabled = true }` | Enforced by EP_AKS_100 and EP_AKS_101 (private cluster, no public FQDN) and IP_AKS_101 (disable local accounts). Host encryption defaults to true for node pools. |
| `policy` | `object` |  | `{ azure_policy_enabled = true, kubernetes_assignments = { enabled = false, definition_ids = [] } }` | Enforced by requirements to keep Azure Policy enabled unless an exception is approved. |






