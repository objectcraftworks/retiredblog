---
layout: page 
title: "ASP.NET/MVC Exception Handling Part 2: ViewData for Error View & its Master/Layout"
comments: true
---
   In [Part1](/blog/2013/07/18/asp-dot-net-exception-handling-explained) of this article, we covered default exception handling of the ASP.NET/MVC framework.
   In this part, We will see how to keep the view data set by the controller for an Error View and master/layout.
 
If you want your Error View use a master that uses ViewData/ViewBag, You will have to set it again in an Error View, which will be a duplicate code.
 
 Let's see how we can display the temperature in every page. 
 To display the temperature in every page, we will have to add to a layout as shown below.

``` html _Layout.cshtml
<Label>@ViewBag.Temperature</Label>&deg; 
``` 

 <!-- more -->
Then, we will have to set the ```ViewBag.Temperature``` somewhere.
There are multiple ways you can set ViewData for master layouts. 
You can set using a base controller, or in an action/result filter, or in actions. We will set it in an action for this example(for production code, this is not a good practice as you will have to set this data in every action).
``` csharp
 public ActionResult Index() {
  ViewBag.Temperature = "67";
  return View();
 }
```
If you run the application, You should see 67&deg; in the page.

Now, let's say some thing happens immediately after ```ViewBag.Temperature``` and causes an exception. Let's throw an exception and run the application.

``` csharp
 public ActionResult Index() {
  ViewBag.Temperature = "67";
  throw new Excepion();
  return View();
 }
```

The temperature wouldn't be displayed and would be just &deg; in the Error View.

What happened?! 

 The MVC framework doesn't keep the viewdata/viewbag set by the controllers. When the framework creates the Error View, it just creates the ViewData object and sets the model.That's why ```ViewBag.Temperature``` is null. 

Now, let's see how we could solve this problem. We could set the temperature again in an Error View. But that will be a duplicate code. We don't want to do that.


Instead of setting it again, we could retain the Controller ViewData/ViewBag, and set on the View even in the event of exception as shown below.
``` csharp
   ViewResult.ViewData = Controller.ViewData;
```
Now we want this piece of code to be called when an exception is raised.
Where do we add this code? 

 > It turns out that ASP.NET MVC provides exception filters exactly for this reason. 

We could write a new exception filter where we can set ViewResult's ViewData.
We will have to register this filter to Global Filters. We need to make sure our filter gets called after the default filter is called.So the order of the filters is important. 

Instead of worrying about the order, we could just extend the HandleError filter, and add this filter to Global Filter.

Let's see how we can extend the HandleError Filter.Let's call our new filter ```HandleErrorWithViewDataAttribute```.

Let's do some simple testing before adding any code.
To make sure Error View is getting the model, let's go ahead use the model in the View by accessing Exception.

``` html /Shared/Error.cshtml 
<span class="redhead">A minor error has occured: @Model.Exception.Message</span>
``` 
 and throw an exception in Index action of the HomeController. If you run the application now, you should see a message.

Now that we have something to tell us that we didn't break any thing. Let's go ahead extend the HandleError filter.

``` csharp 
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, 
                Inherited = true, 
                AllowMultiple = true)]
public class HandleErrorWithViewDataAttribute : HandleErrorAttribute{

}
```

``` csharp
public override void OnException(ExceptionContext filterContext){ }
``` 
Let's call our code that sets the ViewData on ViewResult. We will have to cast Result to ViewResult type.

``` csharp
var result = filterContext.Result as ViewResult;
result.ViewData = filterContext.Controller.ViewData;
```

Now, let's run the application to test our code. We are expecting to have an Error View with the message and the temperature. But you will get a Yellow screen error page. 

What happened?! 
If you debug the application, you will see that model in the Error View is null.


So when we set ``` result.ViewData = filterContext.Controller.ViewData; ``` , we are overriding the ViewData and the model set by the base class HandleError. That should explain why Error didn't receive model.
  
> @Model in the view is actually ViewData.Model. 

Now instead of setting an entire object, we can just add keys that don't exist in the ViewResult. Add following code to your filter. Make sure you remove the code that sets the entire ViewData.

``` csharp
var keysToAddToView = filterContext.Controller
             .Where(item => !result.ViewData.ContainsKey(item.key);

foreach (var item in keysToAddToView)
{
  result.ViewData.Add(item);
}
```
Now, you should see error message, and temperature when you run the appication as shown below.
{%img /images/posts/aspdotnetmvc/error_layout_viewdata.png %}

You can get the full listing for the filter from Github. 
{% gist 6065646 HandleErrorWithViewDataAttribute.cs %}

 In this part, We have extended the HandleError filter to set the viewdata keys on the ViewResult. This enabled us to use a master/layout for an Error View.

---
Last Updated: 08/07/2013

#### Related Articles:

##### Default exception handling works in [Part 1](/articles/dotnet/aspnet/mvc/exceptions/explained)
##### To use a custom model for an Error View in [Part 3](/articles/dotnet/aspnet/mvc/exceptions/custom-model).
#####  To set custom ViewData for Error Views in [Part 4](/articles/dotnet/aspnet/mvc/exceptions/custom-viewdata). 


