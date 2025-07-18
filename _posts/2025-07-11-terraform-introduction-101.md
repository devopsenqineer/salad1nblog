---
layout: post
title: terraform introduction 101
description: tutorial on terraform
summary: terraform
tags: tutorial
minute: 5
lang: en
---

### Terraform 101 - Tutorial

#### Terraform Basics

So first of all Terraform is a Infrastructure as Code tool. You can create your infrastructure as code in many enviroments (e.g. aws, azure, gcp). With this you can improve standards within your company regarding infrastructure. 

In many cases you want to have a fully automated setup of infrastructure, so you invest once your time learning it and then you can easily adopt making for example own modules. 


#### Installation of Terraform

For macOS
Using Homebrew:

```brew tap hashicorp/tap```

```brew install hashicorp/tap/terraform```
To verify installation:

```terraform -v```

For Windows

```choco install terraform```

Option 2: Manual Installation
Go to the Terraform Downloads Page.

Download the Windows ZIP file.

Extract it to a folder (e.g., C:\Terraform).

Add that folder to your System PATH:

Open System Properties → Environment Variables

Edit the Path variable and add C:\Terraform

To verify installation:

```terraform -v```


### 4 Phases

With the command ```terraform init``` you initialize terraform and if you use modules also loads the modules. In this phase it checks if the basic configuration is correct.

With the command ```terraform plan``` you plan your infrastructure changes or creation of the infrastructure. It checks if all ressources are correct.

With the command ```terraform apply``` you verify your plan and with this you start terraform building your infrastructure.

With the command ```terraform destroy``` which is optional you can delete your infrastructure which you deployed before.

### TFState - Mind of your Infrastrucutre

You need to know that after the third phase worked correctly it creates a tfstate file and there will be stored every infrastructure ressource and its the mind of your infrastructure. its the most important file within your configuration because there could be credentials stored etc. so you need to secure this file. In another tutorial i will go deeper into tfstate. 

### Example which you easily follow

#### Prerequisites

You need to install Docker and of course terraform.

#### main.tf 

```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

resource "docker_network" "ollama_network" {
  name = "ollama_network"
}

resource "docker_image" "ollama" {
  name          = "ollama/ollama:latest"
  keep_locally  = false
}

resource "docker_container" "ollama" {
  name    = "ollama"
  image   = docker_image.ollama.image_id
  restart = "unless-stopped"

  ports {
    internal = 11434
    external = 11434
  }

  networks_advanced {
    name = docker_network.ollama_network.name
  }
}

resource "docker_image" "webui" {
  name         = "ghcr.io/open-webui/open-webui:main"
  keep_locally = false
}

resource "docker_container" "webui" {
  name    = "webui"
  image   = docker_image.webui.image_id
  restart = "unless-stopped"

  env = [
    "OLLAMA_BASE_URL=http://ollama:11434"
  ]

  ports {
    internal = 3000
    external = 3000
  }

  networks_advanced {
    name = docker_network.ollama_network.name
  }

  depends_on = [
    docker_container.ollama
  ]
}
``` 

#### Output 

Steps to reproduce:

1. ```terraform init```
2. ```terraform plan```
3. ```terraform apply``` and confirm with ```yes```

```
terraform init

Initializing the backend...
Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0"...
- Installing kreuzwerker/docker v3.6.2...
- Installed kreuzwerker/docker v3.6.2 (self-signed, key ID BD080C4571C6104C)
Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://developer.hashicorp.com/terraform/cli/plugins/signing
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
salad1n@salad1n.dev:/mnt/c/Users/salad1n/terraform docker$ 

terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.ollama will be created
  + resource "docker_container" "ollama" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "ollama"
      + network_data                                = (known after apply)
      + network_mode                                = "bridge"
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "unless-stopped"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + healthcheck (known after apply)

      + labels (known after apply)

      + networks_advanced {
          + aliases      = []
          + name         = "ollama_network"
            # (2 unchanged attributes hidden)
        }

      + ports {
          + external = 11434
          + internal = 11434
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_container.webui will be created
  + resource "docker_container" "webui" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = [
          + "OLLAMA_BASE_URL=http://ollama:11434",
        ]
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "webui"
      + network_data                                = (known after apply)
      + network_mode                                = "bridge"
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "unless-stopped"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + healthcheck (known after apply)

      + labels (known after apply)

      + networks_advanced {
          + aliases      = []
          + name         = "ollama_network"
            # (2 unchanged attributes hidden)
        }

      + ports {
          + external = 3000
          + internal = 3000
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.ollama will be created
  + resource "docker_image" "ollama" {
      + id           = (known after apply)
      + image_id     = (known after apply)
      + keep_locally = false
      + name         = "ollama/ollama:latest"
      + repo_digest  = (known after apply)
    }

  # docker_image.webui will be created
  + resource "docker_image" "webui" {
      + id           = (known after apply)
      + image_id     = (known after apply)
      + keep_locally = false
      + name         = "ghcr.io/open-webui/open-webui:main"
      + repo_digest  = (known after apply)
    }

  # docker_network.ollama_network will be created
  + resource "docker_network" "ollama_network" {
      + driver      = (known after apply)
      + id          = (known after apply)
      + internal    = (known after apply)
      + ipam_driver = "default"
      + name        = "ollama_network"
      + options     = (known after apply)
      + scope       = (known after apply)

      + ipam_config (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
salad1n@salad1n.dev:/mnt/c/Users/salad1n/terraform docker$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.ollama will be created
  + resource "docker_container" "ollama" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "ollama"
      + network_data                                = (known after apply)
      + network_mode                                = "bridge"
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "unless-stopped"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + healthcheck (known after apply)

      + labels (known after apply)

      + networks_advanced {
          + aliases      = []
          + name         = "ollama_network"
            # (2 unchanged attributes hidden)
        }

      + ports {
          + external = 11434
          + internal = 11434
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_container.webui will be created
  + resource "docker_container" "webui" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = [
          + "OLLAMA_BASE_URL=http://ollama:11434",
        ]
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "webui"
      + network_data                                = (known after apply)
      + network_mode                                = "bridge"
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "unless-stopped"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + healthcheck (known after apply)

      + labels (known after apply)

      + networks_advanced {
          + aliases      = []
          + name         = "ollama_network"
            # (2 unchanged attributes hidden)
        }

      + ports {
          + external = 3000
          + internal = 3000
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.ollama will be created
  + resource "docker_image" "ollama" {
      + id           = (known after apply)
      + image_id     = (known after apply)
      + keep_locally = false
      + name         = "ollama/ollama:latest"
      + repo_digest  = (known after apply)
    }

  # docker_image.webui will be created
  + resource "docker_image" "webui" {
      + id           = (known after apply)
      + image_id     = (known after apply)
      + keep_locally = false
      + name         = "ghcr.io/open-webui/open-webui:main"
      + repo_digest  = (known after apply)
    }

  # docker_network.ollama_network will be created
  + resource "docker_network" "ollama_network" {
      + driver      = (known after apply)
      + id          = (known after apply)
      + internal    = (known after apply)
      + ipam_driver = "default"
      + name        = "ollama_network"
      + options     = (known after apply)
      + scope       = (known after apply)

      + ipam_config (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.ollama: Creating...
docker_image.webui: Creating...
docker_network.ollama_network: Creating...
docker_network.ollama_network: Creation complete after 2s [id=da60ed0c036a0db3f237e657ffc4922561a3f363f6670c9eecccc98ba599e3dd]
docker_image.ollama: Still creating... [00m10s elapsed]
docker_image.webui: Still creating... [00m10s elapsed]
docker_image.webui: Still creating... [00m20s elapsed]
docker_image.ollama: Still creating... [00m20s elapsed]
docker_image.webui: Still creating... [00m31s elapsed]
docker_image.ollama: Still creating... [00m31s elapsed]
docker_image.ollama: Still creating... [00m41s elapsed]
docker_image.webui: Still creating... [00m41s elapsed]
docker_image.ollama: Still creating... [00m51s elapsed]
docker_image.webui: Still creating... [00m51s elapsed]
docker_image.webui: Still creating... [01m01s elapsed]
docker_image.ollama: Still creating... [01m01s elapsed]
docker_image.ollama: Still creating... [01m11s elapsed]
docker_image.webui: Still creating... [01m11s elapsed]
docker_image.webui: Still creating... [01m21s elapsed]
docker_image.ollama: Still creating... [01m21s elapsed]
docker_image.ollama: Creation complete after 1m25s [id=sha256:029391db139caf88be3125d86633fde6e713ae8865580df93ff56a5dd49da490ollama/ollama:latest]
docker_container.ollama: Creating...
docker_container.ollama: Creation complete after 1s [id=10e22d9f36d12aba90d7d5b63d52978e835359e78c90e1fcb322d51081014334]
docker_image.webui: Still creating... [01m32s elapsed]
docker_image.webui: Still creating... [01m42s elapsed]
docker_image.webui: Creation complete after 1m51s [id=sha256:21f78cce0c130622c149688a31ee9823112df09ba53b8899ece0dbd97706457cghcr.io/open-webui/open-webui:main]
docker_container.webui: Creating...
docker_container.webui: Creation complete after 1s [id=1edb70dd0fcd63cb40f9c8e64f4f033fff75a92e063abdacdca17a1076e5085e]
```


#### Next Tutorial

In the future i will go more deep into tfstate & creating own modules. 

Thanks for reading!