---
layout: post
author: Craftsman
title: "ASP.NET/MVC Exception Handling Part1: Exception Handling Explained"
date: 2013-07-17 16:07
comments: true
categories:
 - ASP.NET/MVC4
 - DRY
 - OCP
 - MVC
 - ASP.NET/MVC4 Exception Handling
 
---
 ASP.NET MVC framework provides a mechanism to handle exceptions. It's very basic. It doesn't scale very well 
in real world situations. There are atleast two such situations, I ran into.
The framework default exception handling doesn't handle these two situations.  

>  1. If you need a custom model for your Error view. 
>  2. If your Error view needs a master/layout that depends on ViewData set by controllers.  

You will have to duplicate the code in the error view to handle these two situations. Lets see with a mockup.
 {% img /images/posts/aspdotnetmvc/exception_handling_problem.png %}

 Let's see how we would implement the temperature display.As Master/layout is the most obvious place to display, we will have to set the view data again in the error view. This will be a duplicate code.

 <!-- more -->
 If you want to pass contact information based on the customer location, you will have to set it in the view. This will tightly couple the View with the domain logic, undermining MVC phiolosphy of seperation of concerns.
 
  We will develop new filters to handle these situations in this article, and see how we can apply DRY principle, and keep View and Domain loosely coupled.

This article is a four part series. 
 
  We will see in
    1. Part 1, how basic exception handling works. 
    2. Part 2, how to keep the view data set by controllers for Error Views. 
    3. Part 3, how to use a custom model.
    4. Part 4, how to set custom ViewData for ErrorViews. 
          
ASP.NET MVC Framework provides exception handling, and we will see how it works in this part.  (if you are familiar with the framework, you can skip this part.)

> if you want to follow along hands on, You can create a ASP.NET MVC  
 web application. I am using MVC 4 version.

When you create a MVC project, it does two things for error handling purpose 

        1. Creates an error view in shared folder (/Views/Shared/Error.cshtml)
        2. Adds HandleError filter to Global filters (/AppStart/FilterConfig.cs) 

Let's first see the contents of the Error view. You would notice ```@model``` set to HandleErrorInfo. The Framework passes HandleErrorInfo to the view.

``` html Views/Shared/Error.cshtml 
    
    @model System.Web.Mvc.HandleErrorInfo 

    @{
       ViewBag.Title = "Error";
     }
     <hgroup class="title">
     <h1 class="error">Error.</h1>
     <h2 class="error">An error occurred while processing your request.</h2>
     </hgroup>
``` 
 We will talk about HandleErrorInfo type in a bit.

Now, Let's see the contents of the FilterConfig.cs file. You would see HandleErrorAttribute filter is added to Global Filter Collection.

``` csharp AppStart/FilterConfig.cs
 public class FilterConfig
 {
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
       filters.Add(new HandleErrorAttribute()); 
    }
 }
```
 To see whether the default mechanism is working fine, let's throw an exception,as shown below, in the HomeController/Index action.

``` csharp Controllers/HomeController.cs
 public ActionResult Index()
 {
    throw new ArgumentNullException();
    ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";

    return View();
 }
```


Now, if you haven't enabled custom errors, you just need to enable it in the ```Web.Config```, as shown below, to have the framework's default exception mechanism to work.
``` xml web.config
<system.web>
	  <customErrors mode="On"/> 
          <!-- On/RemoteOnly. For this post, I used On -->
</system.web>
```
 Now run the application. You should see the error page.
 
{% img /images/posts/aspdotnetmvc/error_page.png Default Error Page %}
 
 Now that we saw how default exception handling works, 
  >let's summarize the code elements we noticed so far. 

>* HandleError filter
 * HandleErrorInfo model for Error view
 * GlobalFilters 
 * Error view 

We will briefly go through each element, and see what it does.

#### HandleError filter

 The important code element of the framework's default exception handling is HandleErrorAttribute. HandleErrorAttribute is actually an exception filter. There are several types of filters in MVC framework to extend the behavior of the actions. 
Exception filter is one of them. Exception filters are called when an exception is caught by the framework. 
``` csharp    
public class HandleErrorAttribute : FilterAttribute, IExceptionFilter  
```
 The HandleError filter is framework's default exception filter to handle exceptions globally. This is done by adding this filter to Global filters as seen in FilterConfig.cs.

``` csharp AppStart/FilterConfig.cs
filters.Add(new HandleErrorAttribute());
```
 
 The basic behavior of this filter is this:
    creates a ViewResult using given View and Master
    creates a model of type HandleErrorInfo 
    sets the model on ViewData of the ViewResult
    sets the HTTP status to 500
 
 This filter need not to be global. You can handle exception either at an action level or at the controller level. 
 
 If you want to handle a particular exception differently with a custom view and/or master, You can do that too.
 For example, if you want to handle an ArgumentNullException differently for an Index action with different error and/or master, you can do as shown below 

``` csharp  
 [HandleError(ExceptionType=typeof(ArgumentNullException),Order = 1, Master="_ErrorLayout", View="CustomError")]
 public ActionResult Index()
```       

Now that we have seen how this filter works, let's see next important code element.

#### HandleErrorInfo
We have seen in Error.cshtml that ```@model``` is set to ```HandleErrorInfo```. 
HandleErrorInfo has properties for the  Exception that is being handled, and names of Controller and action.

#### Global filters 
 The framework defines scope and order for how the filters are called.
Global scope is last in the order, but gets called for every action.
That's why framework adds the HandleError filter to the Global Filter collection. It doesnot stop you add the filter at action or controller levels.

#### Error View
 The framework generates very basic view. It has access to HandleErrorInfo. Using the model, you can get Exception that is being handled, and the action/controller that caused the exception. 


> We have seen how ASP.NET MVC framework provides exception handling, and how following code elements play the role
    HandleError filter
    HandleErrorInfo model for Error View
    Global Filters to handle exceptions globally
    An error view 


Disclaimer: Code listings used to demonstrate the concepts are not ready for production use. So use at your own discretion. ObjectCraftworks is not responsible for any damages.
~                 
