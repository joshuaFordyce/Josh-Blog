---
title: 'OCI Terraform/OKE project'
description: 'In this Project I worked on setting up automation for Oracle managed kubernetes clusters using terraform'
pubDate: 'Jul 02 2022'
heroImage: '../../assets/images/placeholder-hero.jpg'
category: 'Cloud Infrastructure'
tags: []
---

Intro to OCI’s OKE service
Today, we are going to learn how to automate the deployment of OCI’s OKE which is a managed kubernetes service that allows you to operate production level kubernetes. OCI is Oracle’s cloud infrastructure offering which offers infrastructure as a service, software as a service, and platform as a service for primarily enterprise level customers. OCI’s OKE allows you to ensure reliability and high availability for both the control plane and the worker nodes.

Kubernetes brief

Kubernetes is a powerful platform that helps to manage and orchestrate containerized applications by providing APIs that allow the engineer to deploy, maintain, and scale publications. Containerized application architecture allows engineering teams to develop and deploy their applications by separating them into scalable modules and helps them work in smaller, more agile teams.  Before containers,s engineers typically deployed one application on one virtual machine because of the difficulties associated with different dependency versions. Containers allow us to isolate the dependencies of an application from other containers running on the same environment. This allows us to run several different applications on one virtual machine instead of  The fundamental architecture of a Kubernetes Cluster is very complex so we will focus on simplifying it. Essentially we have worker nodes and master node. The master node is also known as the control plane that facilitates the worker node by scheduling, monitoring, and starting/restarting the nodes/pods.





Terraform brief
While we can simply use the OCI console to set up  and define kubernetes clusters, this article will focus on using terraform to automate the provision and management of kubernetes. Terraform configuration is easy to understand, simplifies the management of multi-tier infrastructure, and works well with both cloud infrastructure and on-prem infrastructure. 
Terraform allows us to keep track of our infrastructure in a state file and utilizes this state file as the most correct current state of your infrastructure. We can easily use version control with terraform to collaborate across our team since the configuration is written in a file.



Basic Terraform workflow
Terraform Init initializes the working directory, initializes the downloads and installs the provider’s plugin so that it can be used later. The write command allows us to define the resources we need and the changes we want to make to them. The terraform plan command compares the planned terraform state with the current state. This command allows us to check whether the execution plan includes changes from the current state to the desired state. The terraform apply command actually applies the changes in the execution plan to achieve the desired state of the configuration.




Getting our hands dirty
Now we can begin the actual development. First we need to prepare by installing Terraform, create RSA keys, Add List Policies and Gather the required information:

We can check the terraform version utilizing the terraform -v then we create a directory for our terraform scripts. Next we have to generate RSA keys using the openssl rsa and openssl genrsa commands. Finally, we need authentication information to authenticate the terraform scripts. This information consists of the region , Tenancy OCID, User OCID and Fingerprint.
Finally,  we will focus on automating a simple OKE cluster in one region. Our Architecture will look like this 

In this section we will create the provider-tf directory and create a file call ad.tf. The following code snippet shows how to do this
Mkdir provider-tf 
cd provider-tf
Touch ad.tf


The ad.tf script will allow us to grab the availability domains from our current compartment. Availability domains are a set of data centers within an OCI region . A region can have multiple ADs with separate support resources. The domains are typically connected via a low latency network to allow for fast inter-region data transmission. 
The following code snippet has the actual terraform code to grab a list of  availability domains from our compartments.
data "oci_identity_availability_domains" "ads" {
	compartment_id = "ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
}


Now we can start with creating the VCN that our OKE cluster is going to reside in.
A VCN is a private network that you set up in Oracle data centers. It has subnets, firewall rules, internet gateway route rules, local peering gateways, route tables and more. 

#source from https://registry.terraform.io/modules/oracle-terraform-modules/vcn/oci/
module "vcn"{
  source  = "oracle-terraform-modules/vcn/oci"
  version = "3.1.0"
  # insert the 5 required variables here

  # Required Inputs
  compartment_id = "ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
  region = "us-ashburn-1"

  internet_gateway_route_rules = null
  local_peering_gateways = null
  nat_gateway_route_rules = null

  # Optional Inputs
  vcn_name = "vcn-module"
  vcn_dns_label = "vcnmodule"
  vcn_cidrs = ["10.0.0.0/16"]
  
  create_internet_gateway = true
  create_nat_gateway = true
  create_service_gateway = true
} 




Here we set up a Private Security List. A security list is a virtual firewall with ingress and egress rules that allows specific network traffic in and out. 
#source from https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_security_list

resource "oci_core_security_list" "private-security-list"{
	# Required
	compartment_id ="ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
	vcn_id = module.vcn.vcn_id
	# Optional
	display_name = "security-list-for-private-subnet"


egress_security_rules {
	stateless = false
	destination = "0.0.0.0/0"
	destination_type = "CIDR_BLOCK"
	protocol = "all"
 }

ingress_security_rules {
	stateless = false
	source = "10.0.0.0/16"
	source_type = "CIDR_BLOCK"
	# Get protocol numbers from https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml TCP is 6 
	protocol = "6"
	tcp_options {
		min = 22
		max = 22
	}
}

ingress_security_rules {
	stateless = false
	source = "0.0.0.0/0"
	source_type = "CIDR_BLOCK"
	# Get protocol numbers from https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml ICMP is 1  
	protocol = "1"
	# For ICMP type and code see: https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml
      icmp_options {
	type = 3
	code = 4
	}
      } 
ingress_security_rules {
	stateless = false
	source = "10.0.0.0/16"
	source_type = "CIDR_BLOCK"
	# Get protocol numbers from https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml ICMP is 1  
	protocol = "1"
	
	# For ICMP type and code see:https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml
      icmp_options {
		type = 3
	}
	}


}

Here we set up a private subnet. A subnet is a logical division of a virtual cloud network and consists of a range of addresses. This range of addresses is determined by CIDR blocks.
#source from https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_subnet

resource "oci_core_subnet" "vcn-private-subnet"{

	#Required
	compartment_id="ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
	vcn_id = module.vcn.vcn_id
	cidr_block = "10.0.1.0/24"

	route_table_id = module.vcn.nat_route_id
	security_list_ids = [oci_core_security_list.private-security-list.id]
	display_name = "private-subnet"
}



Here we setup a public security list
#Source from https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_security_list

resource "oci_core_security_list" "public-security-list"{

# Required
  compartment_id = "ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
  vcn_id = module.vcn.vcn_id

# Optional
  display_name = "security-list-for-public-subnet"

egress_security_rules {
      stateless = false
      destination = "0.0.0.0/0"
      destination_type = "CIDR_BLOCK"
      protocol = "all" 
  }



ingress_security_rules { 
      stateless = false
      source = "0.0.0.0/0"
      source_type = "CIDR_BLOCK"
      # Get protocol numbers from https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml TCP is 6
      protocol = "6"
      tcp_options { 
          min = 22
          max = 22
      }
    }
  ingress_security_rules { 
      stateless = false
      source = "0.0.0.0/0"
      source_type = "CIDR_BLOCK"
      # Get protocol numbers from https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml ICMP is 1  
      protocol = "1"
  
      # For ICMP type and code see: https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml
      icmp_options {
        type = 3
        code = 4
      } 
    }   
  
  ingress_security_rules { 
      stateless = false
      source = "10.0.0.0/16"
      source_type = "CIDR_BLOCK"
      # Get protocol numbers from https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml ICMP is 1  
      protocol = "1"
  
      # For ICMP type and code see: https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml
      icmp_options {
        type = 3
      } 
    }
}



Here we set up a public subnet

#source from https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_subnet

resource "oci_core_subnet" "vcn-public-subnet"{
	
	#Required
	compartment_id = "ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
	vcn_id = module.vcn.vcn_id
	cidr_block = "10.0.0.0/24"
	
	#Optional
	route_table_id = module.vcn.ig_route_id
	security_list_ids = [oci_core_security_list.public-security-list.id]
	display_name = "public-subnet"




}



Now we can setup our cluster and we can name it Practice Cluster.
resource "oci_containerengine_cluster" "oke-cluster" {
	# Required
	compartment_id = "ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
	kubernetes_version = "v1.25.4"
	name = "PracticeCluster"
	vcn_id = module.vcn.vcn_id

	#Optionals
	options {
		add_ons{
			is_kubernetes_dashboard_enabled = false
			is_tiller_enabled = false
		}
		kubernetes_network_config {
			pods_cidr = "10.244.0.0/16"
			services_cidr = "10.96.0.0/16"
		}
		service_lb_subnet_ids = [oci_core_subnet.vcn-public-subnet.id]

	}
}



Now we can begin to set up our node-pool. A node-pool is a group of nodes within a cluster that all have the same config.



Now that we have our node pool setup lets set up our actual cluster
#source from https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/containerengine_cluster

resource "oci_containerengine_cluster" "oke-cluster" {
	# Required
	compartment_id = "ocid1.compartment.oc1..aaaaaaaaz362qxneyuw7vialsglurkwrr47aa55dmqctpb42tlcppcto5y6q"
	kubernetes_version = "v1.25.4"
	name = "PracticeCluster"
	vcn_id = module.vcn.vcn_id

	#Optionals
	options {
		add_ons{
			is_kubernetes_dashboard_enabled = false
			is_tiller_enabled = false
		}
		kubernetes_network_config {
			pods_cidr = "10.244.0.0/16"
			services_cidr = "10.96.0.0/16"
		}
		service_lb_subnet_ids = [oci_core_subnet.vcn-public-subnet.id]

	}
}


Here we set up our outputs.tf which would allow us to print details about our resources and export the details to variables which we can then use in further terraform scripts
#Output the "list" of all availability domains.
output "all-availability-domains-in-your-tenancy" {
	value = data.oci_identity_availability_domains.ads.availability_domains
}
# Outputs for the vcn module

output "vcn_id" {
	description = "OCID of the VCN that is created"
	value = module.vcn.vcn_id
}

output "id-for-route-table-that-includes-the-internet-gateway"{
	description = "OCID of the internet-route table. This route tablet has an internet gateway to be used for public subnets"
	value = module.vcn.ig_route_id
}

output "id-for-for-route-table-that-includs-the-nat-gateway" {
	description = "OCID of the nat-route table - this route table has a nat gateway to be used for private subnets. This route table also has a service gateway."
	value = module.vcn.nat_route_id
}
output "public-security-list-name" {
	value = oci_core_security_list.public-security-list.display_name
}
output "public-security-list_OCID"{
	value = oci_core_security_list.public-security-list.id
}



output "private-security-list-name" {
	value = oci_core_security_list.private-security-list.display_name
}
output "private-security-list-OCID" {
	value = oci_core_security_list.private-security-list.id
}
output "private-subnet-name" {

	value = oci_core_subnet.vcn-private-subnet.display_name
}

output "private-subnet-OCID"{
	value = oci_core_subnet.vcn-private-subnet.id
}


#Outputs for public subnet

output "public-subnet-name" {
	value = oci_core_subnet.vcn-public-subnet.display_name
}

output "public-subnet-OCID" {
	value = oci_core_subnet.vcn-public-subnet.id
}


Now that we’ve written our terraform scripts we can start running the commands to provision our infrastructure. To provision the cluster, we have to initialize the modules, provider plugins and the backend that we defined in our scripts. Run the following command:

terraform init



Once we’ve ran the init command we can check the configuration file using the following command

terraform validate



We can then run the command to see our plan of the resources we want to create 
terraform plan


And finally we provision the resources using the terraform apply command

terraform apply


In this article we focused on developing terraform scripts that allow us to provision OKE clusters on oracle cloud infrastructure.  However, this is a very basic architecture that doesn’t ensure a highly available and highly resilient system. In the next edition of this series, we will focus on adding redundancy and reliability to our cloud architecture.
