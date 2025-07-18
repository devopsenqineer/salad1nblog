---
layout: post
title: terraform modules 101
description: tutorial on terraform
summary: terraform
tags: tutorial
minute: 10
lang: en
---

### Terraform 101 - Modules

#### Terraform Modules

So in the first post of this series you learned what is terraform, how to install and how to use it. 

This is a more advanced topic. Modules.

You can create some ressources which you can use e.g. more often (several VMs with unique configuration). You can use the ressources which you defined before in a module. So if you written once your code you can use it multiple times. 

#### How to create Modules?

First of all you need to create a folder which is named ```modules``` and then you create your module name also as a folder e.g. ```hyper-v```.

After this you can create your code. Because I dont know you needs i'll show a old example which i created a year ago. Its for hyper-v based machines with a provider which one dude is developing. 

```
terraform {
  required_providers {
    hyperv = {
      source  = "taliesins/hyperv"
      version = "1.2.1"
    }
  }
}


# Doc: https://registry.terraform.io/providers/taliesins/hyperv/latest/docs/resources/machine_instance

resource "hyperv_machine_instance" "instances" {
  name                       = var.vm_display_name
  static_memory              = true
  memory_startup_bytes       = var.ram
  generation                 = 2
  processor_count            = var.processor_count
  smart_paging_file_path     = var.smart_paging_file_path
  snapshot_file_location     = var.snapshot_file_location
  state                      = "Running"
  wait_for_ips_poll_period   = 5
  wait_for_ips_timeout       = 300
  wait_for_state_poll_period = 2
  wait_for_state_timeout     = 120

  dvd_drives {
    controller_location = 1
    controller_number   = 0
  }

  # Configure processor
  vm_processor {
    compatibility_for_migration_enabled               = false
    compatibility_for_older_operating_systems_enabled = false
    enable_host_resource_protection                   = false
    expose_virtualization_extensions                  = false
    hw_thread_count_per_core                          = 1
    maximum                                           = 100
    maximum_count_per_numa_node                       = 32
    maximum_count_per_numa_socket                     = 1
    relative_weight                                   = 100
    reserve                                           = 0
  }

  # Configure integration services
  integration_services = {
    "Guest Service Interface" = true
    "Heartbeat"               = true
    "Key-Value Pair Exchange" = true
    "Shutdown"                = true
    "Time Synchronization"    = var.time_synchronization
    "VSS"                     = true
  }

  # Create a network adaptor
  network_adaptors {
    name                              = "Network Adapter"
    switch_name                       = var.switch_name
    iov_interrupt_moderation          = "Default"
    iov_weight                        = 0
    allow_teaming                     = "Off"
    packet_direct_moderation_count    = 64
    packet_direct_moderation_interval = 1000000
    vmmq_enabled                      = true
    wait_for_ips                      = false
  }
  hard_disk_drives {
    path                = var.vhd
    controller_number   = 0
    controller_location = 0
    controller_type     = "Scsi" #Choose between SCSI or IDE
  }
  lifecycle {
    ignore_changes = [name, vm_firmware]
  }
}

resource "time_sleep" "just_wait" {
  depends_on      = [hyperv_machine_instance.instances]
  create_duration = "3m"
}

#Doc: https://developer.hashicorp.com/terraform/language/resources/terraform-data
resource "terraform_data" "script_copy" {

  provisioner "file" {
    source      = "${path.module}/files"
    destination = "C:/salad1n/"
    connection {
      type     = var.host_connection.type
      user     = var.host_connection.user
      password = var.host_password
      host     = var.host_connection.host
      port     = var.host_connection.port
      https    = var.host_connection.https
      insecure = var.host_connection.insecure
      use_ntlm = var.host_connection.use_ntlm
    }
  }
  depends_on = [time_sleep.just_wait]
}



#Doc: https://developer.hashicorp.com/terraform/language/resources/terraform-data
resource "terraform_data" "script_exec_ensure_winrm" {

  provisioner "remote-exec" {
    inline = [
      "powershell.exe -ExecutionPolicy Bypass -Command \". C:\\salad1n\\ensure_winrm.ps1 -User '${nonsensitive(var.client_user)}' -Password '${nonsensitive(var.client_password)}' -vm_display_name '${var.vm_display_name}'\""
    ]
    connection {
      type     = var.host_connection.type
      user     = var.host_connection.user
      password = var.host_password
      host     = var.host_connection.host
      port     = var.host_connection.port
      https    = var.host_connection.https
      insecure = var.host_connection.insecure
      use_ntlm = var.host_connection.use_ntlm
    }
  }
  depends_on = [terraform_data.script_copy]
}

#Doc: https://developer.hashicorp.com/terraform/language/resources/terraform-data
resource "terraform_data" "script_exec_ip" {

  provisioner "remote-exec" {
    inline = [
      "powershell.exe -ExecutionPolicy Bypass -Command \". C:\\salad1n\\ipconfig.ps1 ; Set-IP -User '${nonsensitive(var.client_user)}' -Password '${nonsensitive(var.client_password)}' -vm_display_name '${var.vm_display_name}' -IP '${var.IP}' -MaskBits '${var.MaskBits}' -Gateway '${var.Gateway}'\""
    ]
    connection {
      type     = var.host_connection.type
      user     = var.host_connection.user
      password = var.host_password
      host     = var.host_connection.host
      port     = var.host_connection.port
      https    = var.host_connection.https
      insecure = var.host_connection.insecure
      use_ntlm = var.host_connection.use_ntlm
    }
  }
  depends_on = [terraform_data.script_exec_ensure_winrm]
}
#Doc: https://developer.hashicorp.com/terraform/language/resources/terraform-data
resource "terraform_data" "script_exec_host" {

  provisioner "remote-exec" {
    inline = [
      "powershell.exe -ExecutionPolicy Bypass -Command \". C:\\salad1n\\changeHostname.ps1 ; Set-Hostname -User '${nonsensitive(var.client_user)}' -Password '${nonsensitive(var.client_password)}' -vm_display_name '${var.vm_display_name}' -vm_hostname '${var.vm_hostname}'\""
    ]
    connection {
      type     = var.host_connection.type
      user     = var.host_connection.user
      password = var.host_password
      host     = var.host_connection.host
      port     = var.host_connection.port
      https    = var.host_connection.https
      insecure = var.host_connection.insecure
      use_ntlm = var.host_connection.use_ntlm
    }
  }
  depends_on = [terraform_data.script_exec_ip]
}

#Doc: https://developer.hashicorp.com/terraform/language/resources/terraform-data
resource "terraform_data" "script_exec_winrm" {

  provisioner "remote-exec" {
    inline = [
      "powershell.exe -ExecutionPolicy Bypass -File \"C:\\salad1n\\winrm.ps1\""
    ]
    connection {
      type     = var.host_connection.type
      user     = var.host_connection.user
      password = var.host_password
      host     = var.host_connection.host
      port     = var.host_connection.port
      https    = var.host_connection.https
      insecure = var.host_connection.insecure
      use_ntlm = var.host_connection.use_ntlm
    }
  }
  depends_on = [terraform_data.script_exec_host]
}
```

That is my Code for creating a hyper V based machine like windows server 2025. It used the provider from taliesins. For more information you can visit https://registry.terraform.io/providers/taliesins/hyperv/latest/docs. 

Let me know, If you want a full example with CI/CD Deployment. 

### How to use the module now?

I'll show you now how you reference the module which you created.

```
terraform {
  required_providers {
    hyperv = {
      source  = "taliesins/hyperv"
      version = "1.2.1"
    }
  }
}

resource "hyperv_network_switch" "this" {
  name                                    = "vswitch01"
  default_flow_minimum_bandwidth_absolute = 100000000
  default_queue_vmmq_enabled              = true
  default_queue_vrss_enabled              = true
  minimum_bandwidth_mode                  = "Absolute"
  net_adapter_names                       = [var.adapter]
  switch_type                             = var.type
}

locals {
  path_base_vhd = "C:\\salad1n\\#REPLACE#\\Virtual Hard Disks\\#REPLACE#.vhdx"
}

module "dc" {
  host_connection      = var.host_connection
  host_password        = var.userpassword
  source               = "./modules/hyperv_instance"
  vm_display_name      = var.hostname_dc
  vm_hostname          = var.hostname_dc
  vhd                  = replace(local.path_base_vhd, "#REPLACE#", var.hostname_dc)
  switch_name          = hyperv_network_switch.this.name
  IP                   = var.ip_dc
  Gateway              = var.gateway_dc
  MaskBits             = var.mask_bits_dc
  time_synchronization = false
  depends_on           = [module.vmtemplate]
}
```

You reference the module with module. Then you name it like I did with ```module "dc"```. 
After that you specify also the source ```source = "./modules/hyperv_instance"```. If you specify the source you can load the content of your module. While doing this you need to declare also variables in the variables.tf. The variables which should be configured shouldn't have a default value in the variables.tf. This is important to know.


### Questions?

This was the second part of my series terraform 101. 

Did you like it? Have you questions? Should I do also YouTube Videos? Let me know!

Thanks for reading!