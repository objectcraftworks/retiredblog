---
layout: page
title: "ASP.NET/MVC Exception Handling Part 4: ViewDataProvider for Error View"
comments: true
---
 As we have seen so far in this multi part article, the framework creates a ViewData with a model of type HandleErrorInfo and assigns this ViewData object to View. Also, We have seen in [part2](/blog/2013/07/18/asp-dot-net-mvc4-global-error-handling) and [part3](/blog/2013/07/23/custom-model-for-errorview), how we can extend the default exception filter to keep the viewdata items set by controller and to use a custom model.  In this part,  we will see how we can extend the default filter to set the view data. We will use the techniques covered so far.
 
 <!-- more -->
 
  The MVC framework keeps the model in a dictionary called ```ViewDataDictionary```.
You can add view specific data as items to the dictionary.
You can either directly access these items as you would in a dictionary, or use ViewBag, a dynamic object, which gives the items in the dictionary as properties.

  ViewData items and its model property actually constitutes ViewData/ViewModel for the view.  So instead of just having custom model & model provider, we will extend the default filter to take ViewDataProvider. This will allow us set not only custom model, but also view data items.

``` csharp   
[HandleErrorWithCustomViewData(View="ErrorWithCustomModel", 
                               ViewDataProvider=typeof(ErrorViewDataProvider))]
```

  Now, we need to define a contract for ViewDataProvider. 
As Filter has in its hand: ErrorContext, and the default model it creates. Lets make them available to our custom view data provider. Let's define the signature,as shown below.

``` csharp contract 
public interface IErrorViewDataProvider
{
    ViewDataDictionary GetViewData(ExceptionContext context, HandleErrorInfo defaultModel);
}
```

We can implement it as shown below. This particular implementation uses a custom model, and adds the view data items from the controller.
 
``` csharp ErrorViewDataProvider.cs 
public class ErrorViewDataProvider : IErrorViewDataProvider
{
    public ViewDataDictionary GetViewData(ExceptionContext context, HandleErrorInfo defaultModel)
    {
        var model = GetModel(context, defaultModel);
        var viewData = new ViewDataDictionary(model);
        AddControllerViewDataItemsToView(context.Controller.ViewData, viewData);
        return viewData;
    }

    private void AddControllerViewDataItemsToView(ViewDataDictionary controllerViewData, ViewDataDictionary viewViewData)
    {
        var itemsToAdd = controllerViewData
                                        .Where(item => !viewViewData.ContainsKey(item.Key));
        foreach (var item in itemsToAdd)
        {
            viewViewData.Add(item);
        }
    }

    object GetModel(ExceptionContext context, HandleErrorInfo defaultModel)
    {
        return new CustomModel
        {
            Exception = context.Exception,
            SupportInfo = GetSupportInfo(context)
        };
    }

    SupportInfo GetSupportInfo(ExceptionContext context)
    {
        //You can load it from external source using the available context
        return new SupportInfo
        {
           ContactNo = "1800Support"
        };
    }
}
```
Now that we have a viewdata provider, let's extend the default filter to use this this provider. 

``` csharp HandleErrorWithCustomViewDataAttribute.cs 
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true, Inherited = true)]
public class HandleErrorWithCustomViewDataAttribute : HandleErrorAttribute
{
    public Type ViewDataProvider { get; set; }

    public override void OnException(ExceptionContext filterContext)
    {
        if (filterContext.ExceptionHandled == true)
            return;
        base.OnException(filterContext);

        if (filterContext.ExceptionHandled == false)
            return;

        var result = filterContext.Result as ViewResult;
        if (result == null)
            return;
        var viewDataProvider = Activator.CreateInstance(ViewDataProvider) as IErrorViewDataProvider;
        if (viewDataProvider == null)
            return;
        result.ViewData = viewDataProvider.GetViewData(filterContext, result.Model as HandleErrorInfo);
    }
}



```

Let's add this filter to Home/Index as shown below.

```  csharp HomeController.cs
HandleErrorWithCustomViewData(View="ErrorWithCustomModel", 
                              ViewDataProvider = typeof(ErrorViewDataProvider))]
public ActionResult Index()
{
     ViewBag.Temperature = "67";
     throw new ArgumentNullException();
     ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
    return View();
}
```

if you run the application now, you should see Custom Error View with custom model, displaying ContactNo correctly from model.

{% img /images/posts/aspdotnetmvc/error_viewdata_with_custom_model.png %}

   

We have seen how we can use filters to extend the ASP.NET/MVC framework and handle exceptions in MVC way, and keep our code DRY.  
>  * In Part1, We covered the default exception handling mechanism
 * In Part2, We extended default error filter to pass the View Data items set by a controller to Error View
 * In Part3, We wrote an exception filter by extending the default filter to use Custom model and set the model for the error view.
 * In Part4, We extended what learned so far, and provided a way to set the custom view data provider for the error view.
  
---
Last Updated: 08/07/2013

#### Related Articles:

##### Default exception handling works in [Part 1](/articles/dotnet/aspnet/mvc/exceptions/explained)
##### To keep the view data items set by the controllers for Error Views in [Part 2](/articles/dotnet/aspnet/mvc/exceptions/keep-viewdata)                      
##### To use a custom model for an error view in [Part 3](/articles/dotnet/aspnet/mvc/exceptions/custom-model).


 
