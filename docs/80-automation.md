# Automation

The last several pages of tutorial information was complex, but should have provided some insight as to what was being configured. 

We don't have to do this manual process every time we create a new project. It makes sense to do it at least *once*, to learn and understand what's going on. 

But for deploying multiple setups, such as if you wanted to implement the project separation suggested in the [last section](60-ongoing-deployments.md), using provisioning automation makes sense. 

[Terraform](https://www.terraform.io/) allows us to automate infrastructure provisioning, and comes with a [Google Cloud Platform Provider](https://www.terraform.io/docs/providers/google/index.html) out of the box. 

---

This tutorial isn't a full Terraform 101, but it should help guide you along the process. 

To start with, you'll need to [install Terraform for your operating system](https://learn.hashicorp.com/terraform/getting-started/install.html). 


To give an overview of how you can use terraform, a basic Cloud Run setup might be as simple as the [provided example](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html): 

```shell,exclude
resource "google_cloud_run_service" "default" {
  name     = "tftest-cloudrun"
  location = "us-central1"

  template {
    spec {
      containers {
        image = "gcr.io/cloudrun/hello"
      }
    }
  }
}
```

This page has other sample provisionings, such as [Cloud Run + Cloud SQL](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html#example-usage-cloud-run-service-sql) setup, or [Allow Unauthenticated](https://www.terraform.io/docs/providers/google/r/cloud_run_service.html#example-usage-cloud-run-service-noauth) for example.

But our setup is a little bit more complex. We've provided the Terraform files in `terraform/`, so navigate there and initialise

```shell,exclude
git clone git@github.com/GoogleCloudPlatform/django-demo-app-unicodex
cd django-demo-app-unicodex/terraform
```

We'll also be using a custom Terraform provider for berglas. We'll need to [install that manually](https://github.com/sethvargo/terraform-provider-berglas#installation), and ensure we [bootstrap our secret bucket](https://github.com/GoogleCloudPlatform/berglas#setup) (if we haven't already).

You'll also have to follow the [Getting Started - Adding Credentials](https://www.terraform.io/docs/providers/google/getting_started.html#adding-credentials) section in order to apply any configurations to your project, saving the path to your configuration in `GOOGLE_CLOUD_KEYFILE_JSON`. 

But, with all this setup, it could just be a case of running:

```shell,exclude
terraform init
terraform apply 
```

This will prompt you for some variables (with details about what's required, see `variables.tf` for the full list), and to check the changes that will be applied (which can be checked without potentially applying with `terraform plan`). 

You'll see we're separating our Terraform process into three major segments: 

 * Enabling the service APIs (which then allows us to)
 * Create the Cloud SQL database (which then allows us to)
 * Create the secrets, permissions, and other components required. 

This separation means we can stagger out setup where core sections that depend on each other are 