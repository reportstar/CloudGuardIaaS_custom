# Check Point CloudGuard Network Security Virtual WAN Terraform deployment for Azure

This Terraform module deploys Check Point CloudGuard Network Security Virtual WAN NVA solution into an existing vWAN Hub in Azure.
As part of the deployment the following resources are created:
- Resource groups
- Azure Managed Application:
  - NVA
  - Managed identity

For additional information,
please see the [CloudGuard Network for Azure Virtual WAN Deployment Guide](https://sc1.checkpoint.com/documents/IaaS/WebAdminGuides/EN/CP_CloudGuard_Network_for_Azure_vWAN/Default.htm)

## Configurations
- Install and configure Terraform to provision Azure resources: [Configure Terraform for Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/terraform-install-configure).

## Usage
- Choose the preferred login method to Azure in order to deploy the solution:
    <br>1. Using Service Principal:
    - Create a [Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal) (or use the existing one) 
    - Grant the Service Principal at least "**Contributor**" and "**User Access Administrator**" permissions to the Azure subscription<br>
    - The Service Principal credentials can be stored either in the terraform.tfvars or as [Environment Variables](https://www.terraform.io/docs/providers/azuread/guides/service_principal_client_secret.html)<br>
    
      In case the Environment Variables are used, perform modifications described below:<br>
      
       a. The next lines in the versions.tf file, in the provider azurerm resource,  need to be deleted or commented:
            
                provider "azurerm" {
                 
                //  subscription_id = var.subscription_id
                //  client_id = var.client_id
                //  client_secret = var.client_secret
                //  tenant_id = var.tenant_id
                
                   features {}
                }
            
        b. In the terraform.tfvars file leave empty double quotes for client_secret, client_id , tenant_id and subscription_id variables:
        
                client_secret                   = ""
                client_id                       = ""
                tenant_id                       = ""
                subscription_id                 = "" 
        
    <br>2. Using **az** commands from a command-line:
    - Run  **az login** command 
    - Sign in with your account credentials in the browser
    - In the terraform.tfvars file leave empty double quotes for client_secret, client_id and tenant_id variables. 
 
- Fill all variables in the /terraform/azure/nva-into-existing-hub/terraform.tfvars file with proper values (see below for variables descriptions).
- From a command line initialize the Terraform configuration directory:

        terraform init
- Create an execution plan:
 
        terraform plan
- Create or modify the deployment:
 
        terraform apply

### terraform.tfvars variables:
 | Name          | Description | Type   | Allowed values                              | Default |
 |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| -------------  | -------------  |
 | **authentication_method** | The authentication method used to deploy the solution                      | string | "Service Principal"; <br/>"Azure CLI";      | n/a        
 |  |  
 | **client_secret** | The client secret value of the Service Principal used to deploy the solution | string |                                             | n/a
 |  |             |        |                                             |  |
 | **client_id** | The client ID of the Service Principal used to deploy the solution         | string |                                             | n/a
 |  |             |        |                                             |  |
 | **tenant_id** | The tenant ID of the Service Principal used to deploy the solution         | string |                                             | n/a
 |  |             |        |                                             |  |
 | **subscription_id** | The subscription ID is used to pay for Azure cloud services                | string |                                             | n/a
 |  |             |        |                                             |  |
 | **resource-group-name** | The name of the resource group that will contain the managed application   | string | Resource group names only allow alphanumeric characters, periods, underscores, hyphens and parenthesis and cannot end in a period| "tf-managed-app-resource-group" |
 |  |             |        |                                             |  |
 | **location** | The region where the resources will be deployed at                         | string | The full list of supported Azure regions can be found at https://learn.microsoft.com/en-us/azure/virtual-wan/virtual-wan-locations-partners#locations  | "westcentralus" |
 |  |             |        |                                             |  |
 | **vwan-hub-name** | The name of the virtual WAN hub that will be created                       | string | The name must begin with a letter or number, end with a letter, number or underscore, and may contain only letters, numbers, underscores, periods, or hyphens                                                         | n/a                                                        |
 |  |             |        |                                             |  |
 | **vwan-hub-resource-group** | The virtual WAN hub resource group name                                           | string | | n/a                                              |
 |  |             |        |                                             |  |
 | **managed-app-name** | The name of the managed application that will be created                   | string | The name must begin with a letter or number, end with a letter, number or underscore, and may contain only letters, numbers, underscores, periods, or hyphens | "tf-vwan-managed-app-nva"                                                        |
 |  |             |        |                                             |  |
 | **nva-name** | The name of the NVA that will be created                                   | string | The name must begin with a letter or number, end with a letter, number or underscore, and may contain only letters, numbers, underscores, periods, or hyphens | "tf-vwan-nva"                                                          |
 |  |             |        |                                             |  |
 | **nva-rg-name** | The name of the resource group that will contain the NVA                   | string | Resource group names only allow alphanumeric characters, periods, underscores, hyphens and parenthesis and cannot end in a period | "tf-vwan-nva-rg"|
 |  |             |        |                                             |  |
 | **os-version** | The GAIA os version                                                        | string | "R8110" <br/> "R8120"   <br/> "R82" | "R8120" |
 |  |             |        |                                             |  |
 | **license-type** | The Check Point licence type                                            | string | "Security Enforcement (NGTP)" <br/> "Full Package (NGTX + S1C)" <br/> "Full Package Premium (NGTX + S1C++)"  | "Security Enforcement (NGTP)" |
 |  |             |        |                                             |  |             |        |                                               |  |
 | **scale-unit** | The scale unit determines the size and number of resources deployed. The higher the scale unit, the greater the amount of traffic that can be handled.          | string | "2"<br/>"4"<br/>"10"<br/>"20"<br/>"30"<br/>"60"<br/>"80"<br/> | "2" |
 |  |             |        |                                             |  |
 | **bootstrap_script** | An optional script to run on the initial boot                              | string | Bootstrap script example: <br/>"touch /home/admin/bootstrap.txt; echo 'hello_world' > /home/admin/bootstrap.txt" <br/>The script will create bootstrap.txt file in the /home/admin/ and add 'hello word' string into it | ""
 |  |             |        |                                             |  |      |  |  |  |
 | **admin_shell** | Enables to select different admin shells                                   | string | /etc/cli.sh; <br/>/bin/bash; <br/>/bin/csh; <br/>/bin/tcsh;   | "/etc/cli.sh" |
 |  |             |        |                                             |  |
 | **sic-key** | The Secure Internal Communication one time secret used to set up trust between the gateway object and the management server                                     | string | Only alphanumeric characters are allowed, and the value must be 12-30 characters long  | n/a                                            |
 |  |             |        |                                             |  |
 | **ssh-public-key** | The public ssh key used for ssh connection to the NVA GW instances         | string | ssh-rsa xxxxxxxxxxxxxxxxxxxxxxxx generated-by-azure; | n/a               |            | string | gateway; <br/>standalone; |
 |  |             |        |                                             |  |
 | **bgp-asn** | The BGP autonomous system number                                           | string | 64512 | "64512" ||
 |  |             |        |                                             |  |
 | **custom-metrics** | Indicates whether CloudGuard Metrics will be use for gateway monitoring    | string | yes; <br/>no; | "yes" |
 |  |             |        |                                             |  |
 | **routing-intent-internet-traffic** | Set routing intent policy to allow internet traffic through the new nva    | string | yes; <br/>no;<br/>Please verify routing-intent is configured successfully post-deployment | "yes" |
 |  |             |        |                                             |  |
 | **routing-intent-private-traffic** | Set routing intent policy to allow private traffic through the new nva     | string | yes; <br/>no;<br/>Please verify routing-intent is configured successfully post-deployment | "yes" |
 |  |         |  |                                             |  |
 | **smart1-cloud-token-a** | Smart-1 Cloud token to connect automatically ***NVA instance a*** to Check Point's Security Management as a Service. <br/><br/> Follow these instructions to quickly connect this member to Smart-1 Cloud - [SK180501](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk180501) | string | A valid token copied from the Connect Gateway screen in Smart-1 Cloud portal  | n/a                                                   |  |
 |  |             |  |                                             |  |
 | **smart1-cloud-token-b** | Smart-1 Cloud token to connect automatically ***NVA instance b*** to Check Point's Security Management as a Service. <br/><br/> Follow these instructions to quickly connect this member to Smart-1 Cloud - [SK180501](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk180501) | string | A valid token copied from the Connect Gateway screen in Smart-1 Cloud portal  | n/a                                                   |  |
 |  |             |  |                                             |  |
 | **smart1-cloud-token-c** | Smart-1 Cloud token to connect automatically ***NVA instance c*** to Check Point's Security Management as a Service. <br/><br/> Follow these instructions to quickly connect this member to Smart-1 Cloud - [SK180501](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk180501) | string | A valid token copied from the Connect Gateway screen in Smart-1 Cloud portal  | n/a                                                   |  |
 |  |             |  |                                             |  |
 | **smart1-cloud-token-d** | Smart-1 Cloud token to connect automatically ***NVA instance d*** to Check Point's Security Management as a Service. <br/><br/> Follow these instructions to quickly connect this member to Smart-1 Cloud - [SK180501](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk180501) | string | A valid token copied from the Connect Gateway screen in Smart-1 Cloud portal  | n/a                                                   |  |
 |  |             |  |                                             |  |
 | **smart1-cloud-token-e** | Smart-1 Cloud token to connect automatically ***NVA instance e*** to Check Point's Security Management as a Service. <br/><br/> Follow these instructions to quickly connect this member to Smart-1 Cloud - [SK180501](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk180501) | string | A valid token copied from the Connect Gateway screen in Smart-1 Cloud portal   | n/a                                                  |  |
 |  |  

## Conditional creation
-  To enable CloudGuard metrics in order to send statuses and statistics collected from the gateway instance to the Azure Monitor service:
  ```
  custom-metrics = yes
  ```

## Example
    authentication_method           = "Service Principal"
    client_secret                   = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    client_id                       = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    tenant_id                       = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    subscription_id                 = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    resource-group-name             = "tf-managed-app-resource-group"
    location                        = "westcentralus"
    vwan-hub-name                   = "tf-vwan-hub"
    vwan-hub-resource-group         = "tf-vwan-hub-rg"
    managed-app-name                = "tf-vwan-managed-app-nva"
    nva-rg-name                     = "tf-vwan-nva-rg"
    nva-name                        = "tf-vwan-nva"
    os-version                      = "R8120"
    license-type                    = "Security Enforcement (NGTP)"
    scale-unit                      = "2"
    bootstrap-script                = "touch /home/admin/bootstrap.txt; echo 'hello_world' > /home/admin/bootstrap.txt"
    admin-shell                     = "/etc/cli.sh"
    sic-key                         = "xxxxxxxxxxxx"
    ssh-public-key                  = "ssh-rsa xxxxxxxxxxxxxxxxxxxxxxxx imported-openssh-key"
    bgp-asn                         = "64512"
    custom-metrics                  = "yes"
    routing-intent-internet-traffic = "yes"
    routing-intent-private-traffic  = "yes"
    smart1-cloud-token-a            = ""
    smart1-cloud-token-b            = ""
    smart1-cloud-token-c            = ""
    smart1-cloud-token-d            = ""
    smart1-cloud-token-e            = ""   
    existing-public-ip              = ""
    new-public-ip                   = "yes"

## Known limitations
1. 'terraform destroy' doesn't work if routing-intent is configured. To destroy the deployment, the routing-intent should be deleted manually first. 

## Revision History
In order to check the template version refer to the [sk116585](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk116585)

| Template Version | Description       |
|------------------|-------------------|
| 20241028         |Added R82 version support                   |
| 20240613         | Cosmetic fixes & default values |
| 20240228         | Added public IP for ingress support  |                                                              | |
| 20231226         | First release of Check Point CloudGuard Network Security Virtual WAN Terraform deployment for Azure | |


## License

See the [LICENSE](../../LICENSE) file for details

