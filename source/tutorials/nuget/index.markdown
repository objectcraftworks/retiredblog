---
layout: page
title: "nuget"
date: 2013-07-08 20:24
comments: true
sharing: true
footer: true
---
 In this tutorial, We will be covering every day use of Nuget, such as 

* Install/Update/Remove  a package
* Managing solution wide packages
* Install/Update a particular version of a package
* Assembly Binding Redirect

   We will be using two interfaces of Nuget - Nuget Dialog & Package Manager Powershell Console.

  In this tutorial, we will use a solution with two asp.net mvc4 projects. We will try to manage jQuery for these projects.

## What's Nuget?
  Without Nuget, you would be doing several things to install a package say for example NHibernate. You will first google it for the download link,
and download it, and install it or unzip it, and copy the necessary assemblies to your lib folder where you maintain all dependent assemblies.you might have to add configuration settings to your web.config/app.config 


 You will be repeating more or less similar steps to upgrade a package. 

 Removing a package becomes tricky, and if it's a old,legacy package, it gets harder. If you look at any legacy enterprise .NET visual studio solutions, You will find plent of references to unused assemblies and their configuration settings. It takes a heroic effort to clean up with out breaking changes.

  Good news is that Nuget makes aforementioned process easy. Nuget is a visual studio extension.Nuget provides a central repository, where package authors publish their packages. Nuget also provides client tools to find, download, and install/update/remove packages.   
 
 What's Nuget package? Short answer is it's similar to a install script. Nuget package bundles both actions to be taken when installing/unstalling and the assemblies, and other necessary files.

#### Install a package using Nuget Dialog


  1. Right click on the project in the solution explorer, and then
click on **Manage NuGet Packages** on the menu
{% img /images/tutorials/nuget/open_nuget_dialog.png Open Nuget Dialog %} 


2. It opens the Nuget dialog 

{% img /images/tutorials/nuget/nuget_dialog.png Nuget Dialog %} 




 

