---
layout: page
title: "Managing Packages using NuGet Dialog"
comments: true
sharing: true
footer: true
---
 
We will see how to manage nuget packages using NuGet Dialog. 

## Contents
<div>
<div><a href="#install-using-dialog">Install a package</a></div>
<div><a href="#installed-packages-using-dialog">Find installed packages</a></div>
<div><a href="#uninstall-package-using-dialog">Uninstall a package</a></div>
<div><a href="#update-package-using-dialog">Update a package</a></div>
<div><a href="#manage-solutionwide-packages-using-dialog">Manage packages in a solution</a></div>
<div><a href="#get-package-particular-version"> Install/Update a particular version of a package</a></div>
</div>



  In this tutorial, we will use a visual studio solution with two asp.net mvc4 projects. 

## What's Nuget?


 Without Nuget, you would be doing several things to install a package say for example NHibernate. You will first google it for the download link, and download it, and install it or unzip it, and copy the necessary assemblies to your lib folder where you maintain all dependent assemblies.you might have to add configuration settings to your web.config/app.config

You will be repeating more or less similar steps to upgrade a package.

Removing a package becomes tricky, and if it’s a old,legacy package, it gets harder. If you look at any legacy enterprise .NET visual studio solutions, You will find several references to unused assemblies and their configuration settings. It usually takes a heroic effort to clean up dead code with out breaking changes.

Good news is that Nuget makes aforementioned process easy. Nuget is a visual studio extension.Nuget provides a central repository, where package authors publish their packages. Nuget also provides client tools to find, download, install, update, and remove packages.

What’s a Nuget package? Short answer is it’s similar to a install script. Nuget package bundles both actions to be taken when installing/uninstalling, assemblies, and other necessary files.

Let’s create a asp.net mvc project call it App. By default, Asp.net mvc 4 project adds several nuget packages, including jQuery 1.8.4 version.


<div id="install-using-dialog"></div>
##  Install a package using Nuget Dialog

We will try to add log4Net.

  1. Right click on the project in the solution explorer, and then
   click on **Manage NuGet Packages** on the menu
   {% img /images/tutorials/nuget/open_nuget_dialog.png Open Nuget Dialog %} 


  2. It opens the Nuget dialog 
   {% img /images/tutorials/nuget/nuget_dialog.png Nuget Dialog %} 


  3. Search for a package. Type log4Net in search box, and hit enter. 
   {% img /images/tutorials/nuget/search_package.png Nuget Dialog %} 

  4. You should see search results in the dialog. Select jQuery package, and click on install.
   {% img /images/tutorials/nuget/install_package.png Install using Nuget Dialog %} 
  
  5. Once installation is done, you should see reference to log4net in your project. 
     {% img /images/tutorials/nuget/observe_package_install.png package install observation%}
   
  
>  Congratulations. You just installed a nuget package using Nuget Dialog.
    
In a Nutshell, To install a package using Nuget Dialog,
      1. Bring up the context menu by right clicking on the project
      2. Click on Manage Nuget Packages
      3. Search the package, and install it. 


<div id="installed-packages-using-dialog"></div>

##  Find installed packages in the project
  
1.Open Nuget Dialog, and click installed packages. You can view all, or search a particular package as we did while installing log4net package. 
     {% img /images/tutorials/nuget/installed_packages.png package install observation%}
  >   If you are wondering why ASP.NET MVC 4 project has so many installed packages, these packages are added out of the box by asp.net mvc 4 project template.
      
      1. Bring up Nuget dialog
      2. Click on Installed packages
      3. You can see all installed packages or search for a particular package

<div id="uninstall-package-using-dialog"></div>
## Uninstall a package
  1. Bring up Nuget Dialog, click installed packages, and search for log4net.
     You should see log4net in the search results.Click uninstall button.
     {% img /images/tutorials/nuget/uninstall_package_dialog.png package install observation%}
  2. To test our action, let's verify that log4net is removed from the references.
 
<div id="update-package-using-dialog"></div>
## Update a package 
  For this, we will update jQuery. ASP.NET MVC 4 adds jQuery 1.8.2 when creating the project. We will update jQuery using Nuget and see what happens.

  1. Bring up Nuget Dialog, click updates. if you don't see jQuery, search.
     You should see jQuery in the search results.Click update button.
     {% img /images/tutorials/nuget/update_package_dialog.png package update%}
 
  2. Let's check whether jQuery version is updated. It updates in Scripts folder. 
     {% img /images/tutorials/nuget/update_package_observation.png package update%}

  Now, What's official package source? 
 
> Nuget uses a central repository, where package can be published. When you are either searching online, or looking for updates, Nuget checks in the central repository. That repository is called official package source. 

     1. Bring up the Nuget dialog, click on updates 
       * search if it's not visible in the list. 
     2. Select the package and click on update button



<div id="manage-solutionwide-packages-using-dialog"></div>
## Manage packages in a solution

For this task, we will have to add an another project. Let's go ahead and create a ASP.NET MVC 4 project. Let's call it AppAdmin.
    
  1. This time, let's bring up the context menu by right clicking on the solution.
 {% img /images/tutorials/nuget/manage_packages_for_solution.png Manage packages for solution %}

  2.Let's search for jQuery in the installed packages
 {% img /images/tutorials/nuget/manage_sln_by_search_using_dialog.png Manage packages for solution %}
  
  3. Select jQuery version 2.0.2 and click Manage button   
 {% img /images/tutorials/nuget/mgm_sln_update.png Manage packages for solution %}
  
  3. Select Projects dialog will popup for us to select/deselect projects to install/uninstall packages.   
 {% img /images/tutorials/nuget/mgm_sln_selected_project.png Manage packages for solution %}
 
  4. Let's go ahead, and select AppAdmin project. If you are expecting AppAdmin gets latest jQuery version. That's correct.
 
{% img /images/tutorials/nuget/mgm_sln_after_update.png Manage packages for solution %}
{% img /images/tutorials/nuget/mgm_sln_update_pkg_observation.png Manage packages for solution %}

#### Remove a package from all the projects in the solution
{% img /images/tutorials/nuget/remove_all_projects_dialog.png Manage packages for solution %}

<div id="get-package-particular-version"></div>
## Installing/Updating a particular version 

   You can't install or update using the Nuget Dialog. You will have to use the Package Manager Console to install or update a particular version of a package.
If you want to install/update a particular version, and/or manage packages using Package Manager Console, there is an [article](using-console/) covering it in detail. 


## Wrap up

  We have seen how to install, update, remove a package using NuGet Dialog. NuGet Dialog provides interface to manage packages in a solution. We learned that NuGet Dialog doesn't provide any interface to install or update  a particular version of a package. NuGet package authors and users use a central repository to publish and discover packages. 
  
  NuGet definitely simplifies the package management for .NET solutions. There is a room for improvement in NuGet, but for now, we can do most of the basic things with NuGet.


---
Last Updated: 07/27/2013

#### Related Articles
##### [Managing NuGet Packages using NuGet Console](using-console)
  
