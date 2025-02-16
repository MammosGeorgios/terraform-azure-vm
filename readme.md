# Azure VM With Terraform

## Getting started with Terraform

### Instal terraform and azure-cli on macos with brew

- Instal terraform from a terminal with `brew update && install terraform`
- Verify installation with `terraform -help'
- Instal azure-cli from the terminal with `brew install azure-cli`
- Verify installation with `az -help'

### Choose subscription, prepare service principal and credentials

- From terminal run `az login`. A browser will open and you will need to login properly. Once successful, you will receive a list of all your subscriptions
- Next we want to set subscription with `az account set --subscription "subscription_id_from_last_window` (This part can be skipped if you chose the correct subscription in the previous step, keeping the command in case you want to switch without having to login again)
- Run`az ad sp create-for-rbac --role="Contributor" --scopes="subscriptions/subscription_id_from_last_window"` and keep the results handy somewhere, as we will need them afterwards

### Setting up terraform files

If you are using my exact setup then we only need to do add the service principal credentials  in in our setup and we good to go. You can either create a new file, or uncomment and update the main. I recommend creating a few file named **azurerm.tf** and setting up your credentials there (the file is set up to be ignored by git from the get-go).

``` terraform
provider "azurerm" {
  features {}

  client_id       = "app_id_goes_here"
  client_secret   = "password_goes_here"
  tenant_id       = "tenant_id_goes_here"
  subscription_id = "subscription_id_goes_here"
}
```

### Create the resources with terraform

Finally time to get our vm going!

- First run `terraform init -upgrade` to initialize/download everything needed. Assuming all is well, you should see some version checks or downloads and a **Terraform has been successfully initialized!** message
- To validate that everything is in place, we use `terraform validate`
- Next we can run `terraform plan` and see what will happen with our setup
- Finally, run `terraform apply`, answer yes once prompted in terminal, and watch the light show as everything is created for us

### Connection to the new VM

After the `terraform apply` command has ended, the final texts in the terminal should inform up on the public IP given to the vm. Also we should have a new file named **ssh_key.pem** available to us.

Assuming you run the command at the same directory as the pem file, we can connect to our VM with the the following (will need to accept the risk of an non-authenticated key):

``` bash
ssh -i ssh_key.pem azureadmin@9.141.160.198
```

If you get an error for the pem file, saying that the file is "too open" or "bad permissions", then first run the following command:

``` bash
chmod 600 ssh_key.pem
```

Assuming everything went well, you should be connected to your VM!

### Closing all resources

To close all resources quickly, you can run the `terraform destroy` command (will need to say yes as well or use `-auto-approve` with the command)

### Other useful Terraform commands

- `terraform fmt` will format your files. really like this one :)
- `terraform show` will show the **state**, giving you an idea of what is up and running or not

### Something broke with terraform, what am i going to do?!?!?!?!?

The following assumes you don;t care about the VM. If that is nto the case, you will need to handle your case specifically

- Delete folder .terraform and files terraform.lock.hcl, terraform.tfstate, terraform.tfstate.backup and run terraform init again
- Delete the resource group from Azure, along with everything inside
