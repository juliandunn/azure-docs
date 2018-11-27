---
title: Azure virtual machine deployment with Chef | Microsoft Docs
description: Learn how to use Chef to do automated virtual machine deployment and configuration on Microsoft Azure
services: virtual-machines-windows
documentationcenter: ''
author: diegoviso
manager: jeconnoc
tags: azure-service-management,azure-resource-manager
editor: ''

ms.assetid: 0b82ca70-89ed-496d-bb49-c04ae59b4523
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-multiple
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: diviso

---
# Automating Azure virtual machine deployment with Chef
[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

Chef is a great tool for delivering automation and desired state configurations.

With the latest cloud API release, Chef provides seamless integration with Azure, giving you the ability to provision and deploy configuration states through a single command.

In this article, you set up your Chef environment to provision Azure virtual machines and walk through creating a policy or “CookBook” and then deploying this cookbook to an Azure virtual machine.

Let’s begin!

## Chef basics
Before you begin, [review the basic concepts of Chef](http://www.chef.io/chef). 

The following diagram depicts the high-level Chef architecture.

![][2]

Chef has three main architectural components: Chef Server, Chef Client (node), and Chef Workstation.

The Chef Server is the management point and there are two options for the Chef Server: a hosted solution or an on-premises solution.

The Chef Client (node) is the agent that sits on the servers you are managing.

The Chef Workstation, which is the name for both the admin workstation where you create policies and execute management commands and the software package of Chef tools.
Generally, you'll see _your workstation_ as the location where you perform actions and _Chef Workstation_ for the software package.
For example, you'll download the knife command as part of _Chef Workstation_, but you run knife commands from _your workstation_ to manage infrastructure.

Chef also the concepts of “Cookbooks” and “Recipes”, which are effectively the policies we define and apply to the servers.

## Preparing your workstation

First, prep your workstation by creating a directory to store Chef configuration files and cookbooks.

Create a directory called C:\chef.

## Setup Chef Server

This guide assumes that you will sign up for Hosted Chef.

If you're not already using a Chef Server, you can:

* Sign up for [Hosted Chef](https://manage.chef.io/signup), which is the fastest way to get started with Chef.
* Install a standalone Chef Server on linux-based machine, following the [installation instructions](https://docs.chef.io/install_server.html) from [Chef Docs](https://docs.chef.io/).

## Creating a hosted Chef account

Sign up for a hosted Chef account [here](https://manage.chef.io/signup).

During the sign-up process, you will be asked to create a new organization.

![][3]

Once your organization is created, download the starter kit.

![][4]

> [!NOTE]
> If you receive a prompt warning you that your keys will be reset, it’s okay to proceed as we have no existing infrastructure configured as yet.
> 
> 

This starter kit zip file contains your organization config files and keys.

## Configuring your Chef workstation

Extract the content of the chef-starter.zip to C:\chef.

Copy all files under chef-starter\chef-repo\.chef to your c:\chef directory.

Your directory should now look something like the following example.

![][5]

You should now have four files including the Azure publishing file in the root of c:\chef.

The PEM files contain your organization and admin private keys for communication while the knife.rb file contains your knife configuration. We will need to edit the knife.rb file.

Open the file in your editor of choice and modify the “cookbook_path” by removing the /../ from the path so it appears as:

    cookbook_path  ["#{current_dir}/cookbooks"]

Also add the following line reflecting the name of your Azure publish settings file.

    knife[:azure_publish_settings_file] = "yourfilename.publishsettings"

Your knife.rb file should now look similar to the following example.

![][6]

These lines will ensure that Knife references the cookbooks directory under c:\chef\cookbooks, and also uses our Azure Publish Settings file during Azure operations.

## Install Chef Workstation

Next, [download and install](https://downloads.chef.io/chef-workstation/) Chef Workstation.

New ad-hoc commands `chef-run` and ChefDK CLI commands such as `chef` are available via Chef Workstation. See your installed version of Chef Workstation and the Chef tools with `chef -v`. You can also check your Workstation version by selecting "About Chef Workstation" from the Chef Workstation App.

`chef --version` should return something like:

```
Chef Workstation: 0.2.29
  chef-run: 0.2.2
  Chef Client: 14.6.47
  delivery-cli: master (6862f27aba89109a9630f0b6c6798efec56b4efe)
  berks: 7.0.6
  test-kitchen: 1.23.2
  inspec: 3.0.12
```

![][7]

Install in the default location of c:\opscode. This install takes around 10 minutes.

Confirm your PATH variable contains entries for C:\opscode\chefdk\bin;C:\opscode\chefdk\embedded\bin;c:\users\yourusername\.chefdk\gem\ruby\2.0.0\bin

If they are not there, make sure you add these paths!

> [!NOTE]
> The order of the path is important! If your opscode paths are not in the correct order you will have issues. 
> 

Reboot your workstation before you continue.

### Install Knife Azurerm

Next, install the Knife Azure extension. This provides Knife with the “Azure Plugin”.

Run the following command.

    chef gem install knife-azure ––pre

> [!NOTE]
> The –pre argument ensures you are receiving the latest RC version of the Knife Azure Plugin which provides access to the latest set of APIs.
> 
> 

It’s likely that a number of dependencies will also be installed at the same time.

![][8]

To ensure everything is configured correctly, run the following command.

    knife azure image list

If everything is configured correctly, you will see a list of available Azure images scroll through.

Congratulations. Your workstation is set up!

## Creating a Cookbook

A Cookbook is used by Chef to define a set of commands that you wish to execute on your managed client. Creating a Cookbook is straightforward, just use the **chef generate cookbook** command to generate the Cookbook template. This cookbook is for a web server that automatically deploys IIS.

Under your C:\Chef directory run the following command.

    chef generate cookbook webserver

This command generates a set of files under the directory C:\Chef\cookbooks\webserver. Next, define the set of commands for the Chef client to execute on the managed virtual machine.

The commands are stored in the file default.rb. In this file, define a set of commands that installs IIS, starts IIS and copies a template file to the wwwroot folder.

Modify the C:\chef\cookbooks\webserver\recipes\default.rb file and add the following lines.

    powershell_script 'Install IIS' do
         action :run
         code 'add-windowsfeature Web-Server'
    end

    service 'w3svc' do
         action [ :enable, :start ]
    end

    template 'c:\inetpub\wwwroot\Default.htm' do
         source 'Default.htm.erb'
         rights :read, 'Everyone'
    end

Save the file once you are done.

## Creating a template

In this step, you'll generate a template file to used as the default.html page.

Run the following command to generate the template:

    chef generate template webserver Default.htm

Navigate to the `C:\chef\cookbooks\webserver\templates\default\Default.htm.erb` file. Edit the file by adding some simple “Hello World” HTML code, and then save the file.

## Upload the Cookbook to the Chef Server
In this step, you make a copy of the Cookbook that you have created on the local machine and upload it to the Chef Hosted Server. Once uploaded, the Cookbook appears under the **Policy** tab.

    knife cookbook upload webserver

![][9]

## Deploy a virtual machine with Knife Azure
Deploy an Azure virtual machine and apply the “Webserver” Cookbook which installs the IIS web service and default web page.

In order to do this, use the **knife azure server create** command.

An example of the command appears next.

    knife azure server create --azure-dns-name 'diegotest01' --azure-vm-name 'testserver01' --azure-vm-size 'Small' --azure-storage-account 'portalvhdsxxxx' --bootstrap-protocol 'cloud-api' --azure-source-image 'a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-Datacenter-201411.01-en.us-127GB.vhd' --azure-service-location 'Southeast Asia' --winrm-user azureuser --winrm-password 'myPassword123' --tcp-endpoints 80,3389 --r 'recipe[webserver]'

The parameters are self-explanatory. Substitute your particular variables and run.

> [!NOTE]
> Through the command line, I’m also automating my endpoint network filter rules by using the –tcp-endpoints parameter. I’ve opened up ports 80 and 3389 to provide access to my web page and RDP session.
> 
> 

Once you run the command, go to the Azure portal to see your machine begin to provision.

![][13]

The command prompt appears next.

![][10]

Once the deployment is complete, you should be able to connect to the web service over port 80, because you opened the port when you provisioned the virtual machine with the Knife Azure command. As this virtual machine is the only virtual machine in this cloud service, connect to it with the cloud service url.

![][11]

As you can see, I got creative with my HTML code.

Don’t forget you can also connect through an RDP session from the Azure portal via port 3389.

I hope this has been helpful! Go and start your infrastructure as code journey with Azure today!

<!--Image references-->
[2]: media/chef-automation/2.png
[3]: media/chef-automation/3.png
[4]: media/chef-automation/4.png
[5]: media/chef-automation/5.png
[6]: media/chef-automation/6.png
[7]: media/chef-automation/7.png
[8]: media/chef-automation/8.png
[9]: media/chef-automation/9.png
[10]: media/chef-automation/10.png
[11]: media/chef-automation/11.png
[13]: media/chef-automation/13.png


<!--Link references-->
