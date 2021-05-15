### GSP345 : Automating Infrastructure on Google Cloud with Terraform: Challenge Lab :-

----------------------------------------------------------------------------------------------------------------------------------------------

### Task 1. Create the configuration files

- Make the empty files and directories in Cloud Shell or the Cloud Shell Editor.
Add the following to the each __variables.tf__ file, and fill in the GCP Project ID:
```yaml
variable "region" {
 default = "us-central1"
}

variable "zone" {
 default = "us-central1-a"
}

variable "project_id" {
 default = "<FILL IN PROJECT ID>"
}
```


Add the following to the **main.tf** file:
```yaml
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region

  zone        = var.zone
}


module "instances" {

  source     = "./modules/instances"

}
```
Run `terraform init` in Cloud Shell in the root directory to initialize terraform.

### Task 2. Import infrastructure
Navigate to Compute Engine > VM Instances. Click on **tf-instance-1**. Copy the Instance ID down somewhere to use later. 
Navigate to Compute Engine > VM Instances. Click on **tf-instance-2**. Copy the Instance ID down somewhere to use later. 

Next, navigate to **modules/instances/instances.tf**. Copy the following configuration into the file:
```yaml
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```
To import the **first** instance, use the following command, using the Instance ID for **tf-instance-1** you copied down earlier.
```bash
terraform import module.instances.google_compute_instance.tf-instance-1 <Instance ID - 1>
```
To import the second instance, use the following command, using the Instance ID for **tf-instance-2** you copied down earlier.
```bash
terraform import module.instances.google_compute_instance.tf-instance-2 <Instance ID - 2>
```
The two instances have now been imported into your terraform configuration. You can now optionally run the commands to update the state of Terraform. Type yes at the dialogue after you run the apply command to accept the state changes.
```bash
terraform plan
terraform apply
```
### Task 3. Configure a remote backend
Add the following code to the **modules/storage/storage.tf** file:
```yaml
resource "google_storage_bucket" "storage-bucket" {
  name          = var.project_id
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
```
Next, add the following to the main.tf file:
```yaml
module "storage" {
  source     = "./modules/storage"
}
```
Run the following commands to initialize the module and create the storage bucket resource. Type **yes** at the dialogue after you run the apply command to accept the state changes.
```bash
terraform init
terraform apply
```
Next, update the **main.tf** file so that the terraform block looks like the following. Fill in your GCP Project ID for the bucket argument definition.
```yaml
terraform {
  backend "gcs" {
    bucket  = "<FILL IN PROJECT ID>"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}
```
Run the following to initialize the remote backend. Type **yes** at the prompt. 
```bash
terraform init
```

### Task 4. Modify and update infrastructure
Navigate to modules/instances/instance.tf. Replace the entire contents of the file with the following:
```yaml
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         = var.zone
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = var.zone
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-3" {
  name         = "tf-instance-3"
  machine_type = "n1-standard-2"
  zone         = var.zone
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```
Run the following commands to initialize the module and create/update the instance resources. Type **yes** at the dialogue after you run the apply command to accept the state changes.
```bash
terraform init
terraform apply
```
### Task 5. Taint and destroy resources
Taint the **tf-instance-3** resource by running the following command:
```bash
terraform taint module.instances.google_compute_instance.tf-instance-3
```
Run the following commands to apply the changes:
```bash
terraform init
terraform apply
```
Remove the **tf-instance-3** resource from the **instances.tf** file. Delete the following code chunk from the file.
```yaml
resource "google_compute_instance" "tf-instance-3" {
  name         = "tf-instance-3"
  machine_type = "n1-standard-2"
  zone         = var.zone
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```
Run the following commands to apply the changes. Type **yes** at the prompt.
```bash
terraform apply
```

### Task 6. Use a module from the Registry
Copy and paste the following into the main.tf file:
```yaml
module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 2.5.0"

    project_id   = var.project_id
    network_name = "terraform-vpc"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-central1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-central1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}
```
Run the following commands to initialize the module and create the VPC. Type **yes** at the prompt.
```bash
terraform init
terraform apply
```
Navigate to modules/instances/instances.tf. Replace the entire contents of the file with the following:
```yaml
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         = var.zone
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "terraform-vpc"
    subnetwork = "subnet-01"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = var.zone
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "terraform-vpc"
    subnetwork = "subnet-02"
  }
}
```
Run the following commands to initialize the module and update the instances. Type **yes** at the prompt.
```bash
terraform init
terraform apply
```

### Task 7. Configure a firewall
Add the following resource to the **main.tf** file and fill in the GCP Project ID:
```yaml
resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
 network = "projects/<PROJECT_ID>/global/networks/terraform-vpc"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}
```

### Congratulations!
