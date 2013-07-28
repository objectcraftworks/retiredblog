---
layout: post
author: Craftsman
title: "ASP.NET/MVC Exception Handling Part 2: Passing ViewData to Error View & its Master/Layout"
date: 2013-07-18 16:07
comments: true
categories:
 ASP.NET/MVC4
---
   In [Part1](/blog/2013/07/18/asp-dot-net-exception-handling-explained) of this article, we covered default exception handling of the ASP.NET/MVC framework.
   In this part, We will see how to keep the view data set by the controller for error view and master/layout.
 
If you want your Error View use a master that uses ViewData/ViewBag, You will have to set it again in Error View, which will be a duplicate code.
 
 Let's see how we can display the temperature in every page a site. 
 To display the temperature in every page, we will have to add to a layout as shown below.

``` html _Layout.cshtml
<Label>@ViewBag.Temperature</Label>&deg; 
``` 

Then, we will have to set the ```ViewBag.Temperature``` somewhere.
There are multiple ways you can set ViewData for master layouts. 
You can set using a base controller, or in an action/result filter, or in actions. Though this is not a good practice to set it in an action for production code as this would result in duplicate code, we will set it in an action for this example.
``` csharp
 public ActionResult Index() {
  ViewBag.Temperature = "67";
  return View();
 }
```
Now you should see 67&deg; in the page, if you run it.

Now let's say some thing happens immediately after ViewBag.Temperature and causes an exception. Let's throw an exception and run the application.

``` csharp
 public ActionResult Index() {
  ViewBag.Temperature = "67";
  throw new Excepion();
  return View();
 }
```

The temperature wouldn't be displayed and would be just &deg; in the error view.

What happened? 

 The MVC framework doesn't keep the viewdata/viewbag set by actions in the event of exceptions. When the framework creates the error view, it just simply sets the model of type HandleErrorInfo, which gives the Error View an access to the exception.

Now, let's see how we could solve this problem. We could set the temperature again in Error View. But that will be a duplicate code. We don't want to do that.


Instead of setting it again, we could retain the Controller ViewData/ViewBag, and set on the View even in the event of exception as shown below.
``` csharp
   ViewResult.ViewData = Controller.ViewData;
```
Now we want this piece of code to be called when an exception is raised.
Where do we add this code? 

 > It turns out that ASP.NET MVC provides exception filters exactly for this reason. 

We could write a new exception filter where we can set ViewResult's ViewData.
We will have to register this filter to Global Filters. We need to make sure our filter gets called after the default filter is called.So the order of the filters is important. Make sure you add this filter before the HandleError filter.

Instead of worrying about the order, we could just extend the HandleError filter, and add this filter to Global Filter.

Let's see how we can extend the HandleError Filter.Let's call our new filter HandleErrorWithViewDataAttribute.

Let's do some simple testing before adding any code.
To make sure Error View is getting the model, let's go ahead use the model in the View by accessing Exception.

``` html /Shared/Error.cshtml 
   <span class="redhead">A minor error has occured: @Model.Exception.Message</span>
``` 
 and throw an exception in Index action of HomeController. If you run the application now, you should see a message.

Now that we have something to tell us that we didn't break any thing. Let's go aahead extend the HandleError filter.

``` csharp
  [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
 public class HandleErrorWithViewDataAttribute : HandleErrorAttribute{

 }
```

``` csharp
        public override void OnException(ExceptionContext filterContext){ }
``` 
Let's call our code that sets ViewData on ViewResult. We will have to cast Result to ViewResult type.

``` csharp
    var result = filterContext.Result as ViewResult;
    result.ViewData = filterContext.Controller.ViewData;
```

Now, let's run the application to test our code.We are expecting to have an error view with the message and the temperature. But you will get a Yellow screen error page. What happened? 
If you debug the application, you will see that model in the error view is null.


So when we set 
``` 
  result.ViewData = filterContext.Controller.ViewData;
``` , we are overriding the ViewData and the model set by the base class HandleError. That should explain why Error didn't receive model.
  
> @Model in the view is actually ViewData.Model. 

Now instead of setting entire object, we can just add keys that don't exist in the ViewResult.Add following code to your filter. Make sure you remove the code tthat sets entire ViewData.

``` csharp
 var keysNeedToSetInView = filterContext.Controller
             .Where(item => !result.ViewData.ContainsKey(item.key);

 foreach (var item in keysNeedToSetInView)
 {
    result.ViewData.Add(item);
 }
```
Now, you should error message, and temperature when you run the appication as shown below.
{%img /images/posts/aspdotnetmvc/error_layout_viewdata.png %}

You can get the full lister for the filter from Github. 
{% gist 6065646 HandleErrorWithViewDataAttribute.cs %}

We override the default behavior of the HandleError filter, and set the viewdata keys on the ViewResult. This enabled us to use a master/layout for an error view.



