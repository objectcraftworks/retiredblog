---
layout: page
title: "Managing Nuget packages using Package Manager Console"
comments: true
sharing: true
footer: true
---
  
 In this tutorial, We will be covering every day use of Nuget using Package Manager Console. Nuget has two client interfaces. You can use NuGet Dialog, an UI, or the Console, which is a powershell console. At this time, You will have to use the console if you want to install or update a particular version of a package. 

## Contents

 <div>   [Find all available versions](#Find all available versions)</div>
 <div>   [Install a particular version](#Install a particular version)</div>
 <div>   [Update to a particular version](#Update to a particular version)</div>
 <div>   [Update to a latest version](#Update to a latest version)</div>
 <div>   [Updating a package updates all projects](#Updating a package updates all projects)</div>
 <div>   [Remove a package](#Remove a package)</div>
 <div>   [Managing packages for a solution](#manage-packages-for-a-solution)</div>
  

 In this tutorial, we will use a visual studio solution with two asp.net mvc4 projects. 


## What's Nuget?
  Without Nuget, you would be doing several things to install a package say for example NHibernate. You will first google it for the download link,
and download it, and install it or unzip it, and copy the necessary assemblies to your lib folder where you maintain all dependent assemblies.you might have to add configuration settings to your web.config/app.config 


 You will be repeating more or less similar steps to upgrade a package. 

 Removing a package becomes tricky, and if it's a old,legacy package, it gets harder. If you look at any legacy enterprise .NET visual studio solutions, You will find several references to unused assemblies and their configuration settings. It usually takes a heroic effort to clean up dead code with out breaking changes.

  Good news is that Nuget makes aforementioned process easy. Nuget is a visual studio extension.Nuget provides a central repository, where package authors publish their packages. Nuget also provides client tools to find, download, install, update, and remove packages.   
 
 What's a Nuget package? Short answer is it's similar to a install script. Nuget package bundles both actions to be taken when installing/uninstalling, assemblies, and other necessary files.

 Let's create a asp.net mvc project call it App. By default, Asp.net mvc 4 project adds several nuget packages, including jQuery 1.8.4 version. 

You can bring the Package Manager(PM) Console up using the menu item *Tools/Library Package Manager / Package Manager Console*. 

{% img /images/tutorials/nuget/pm_console_menu_item.png Installing a particular version  %}

<div id="Find all available versions"></div>
## Find all available versions 
 For this, we will try to install normalize.css package. Normalize.css keeps a web.app consistent across browsers. We can use get-package cmdlet to find the packages. `Get-Package` is a powershell cmdlet. Similar to NuGet Dialog, this cmdlet searches packages in the NuGet's repository. There are two parameters we will use to find Normalize.css and all its versions. They are `-ListAvailable`, and `-AllVersions`. `Get-Package` gets the list of all packages  matching  the search string, and also all the available versions. This gets latest version only, if you omit `-AllVersions` parameters.

  
Let's go ahead, and run
  `Get-Package -ListAvailable -AllVersions normalize.css` in the PM Console.  

{% img /images/tutorials/nuget/find_all_versions_console.png Installing a particular version  %}

Now that we know all available version, let's go ahead install a particular version.
<a id="Install a particular version"></a> 
## Install a particular version 
 Let's say we want to install Normalize.css version 1.1.1. You can use another cmdlet Install-Package by passing the package id which is in this case Normalize.css and the version to the parameter -Version. 
If you don't specify a version, this cmdlet installs latest version available. In this example, that would be 2.1.2. 

Let's go ahead install normalize.css version 1.1.1 by running `install-package Normalize.css -Version 1.1.1`.
 
{% img /images/tutorials/nuget/install_particular_version_console.png Installing a particular version  %}

The console uses default project setting as the project for all cmdlets where `-Project` parameter is available. The cmdlet installed the package in the default project.
As an exercise, let's go ahead install this particular version in other project AppAdmin. 
<a id="Update to a particular version"></a> 
## Update to a particular version 
 We have seen how we can specify version to the cmdlet `install-package` using a parameter `-Version`, we can use same parameter to update a package to a particular version. But this time, there is another cmdlet. You will have to use `update-package`, along with parameter `-Version` to update a package to a pacticular version.  
 
  For this, we will try to update to `version 2.0.1` of Normalize.css package. We have already seen how to find the all available versions using the cmdlet get-package. Using get-package let's find again all available versions of the package Normalize.css. Then let's use cmd `update-package normalize.css -Version 2.0.1` to install 2.0.1 version of the Normalize.css package.
 
{% img /images/tutorials/nuget/updating_particular_version_console.png  updating to a particular version  %}

Now what would `update-package` cmdlet do if you omit -Version? 
<a id="Update to a latest version"></a> 
## Update to a latest version 
 We have just seen how to update to a particular version using the cmdlet `Update-Package` along with its parameter `-Version`. If you want to update to a latest version, then you just have to omit the `-Version` parameter as shown below. 

 Without `-Version`, this would update to latest the version of Normalize.css. 

{% img /images/tutorials/nuget/update_package_console.png  updating to latest version  %}

<a id="Updating a package updates all projects"></a>
## Updating a package updates all projects
As we talked about how NuGet uses the default project selection, and a cmdlet 
uses the default project as a value to -Project. What that means is that all cmds work on that project only, excep the cmdlet `update-package`. If you update-package with out -version, it updates
the package in all projects which have this package installed. 

If you try to update the Normalize.css package, this will update the package in both the projects. 

{% img /images/tutorials/nuget/update_package_updates_sln.png  updating to latest version  %}

So far, we have seen how to install/update lastest version, and particular version using the Package Manager (PM) Console. Now we will try to remove the package we have just added.
<a id="Remove a package"></a> 
## Remove a package 

The cmdlet `Uninstall-Package` removes a package in the default project as shown below.  

{% img /images/tutorials/nuget/uninstall_package_console.png  updating to latest version  %}

<a id="manage-packages-for-a-solution"></a>
## Managing packages for a Solution
PM Console doesn't have a cmdlet to manage packages for a solution. So far we have seen how cmdlets work on the default project setting, except update-package, which updates a package in all project where the package is present. Using piping of two cmdlets, we can manage packages for a .NET solution.
There is cmdlet `get-project` returns a project object which is a handle to IDE's project.

``` powershell
Get-Project -All # Returns all projects
Get-Project  # Returns the default project
Get-Project App, AppAdmin # Returns projects App, and AppAdmin   
```
if you want to remove the package Normalize.css from the both projects App, and AppAdmin, you can pipe both cmdlets get-project and uninstall-project as
``` powershell
 Get-Project App,AppAdmin | uninstall-package Normalize.css
```
Here is how my PM console looks after executing both the cmdlets.

{% img /images/tutorials/nuget/manage_pkgs_sln_console.png  updating to latest version  %}

## Wrap up
We have seen how to install, remove, update a package using the PM Console. You will have to use PM console to install or update a particular version. Nuget Dialog doesn't support installing or updating a particular version. 





* * *
 Last Updated: 08/04/2013
 
#### Related Articles
##### [Managing NuGet Packages using NuGet Dialog](/articles/dotnet/nuget/using-dialog)
