---
layout: post
author: Craftsman
title: "ASP.NET MVC Exception Handling Part 3: Custom Model for Error View"
date: 2013-07-23 16:07
comments: true
categories:
 ASP.NET/MVC4
---
 
In [Part1](/blog/2013/07/18/asp-dot-net-exception-handling-explained) of this article, we covered default exception handling of the ASP.NET/MVC framework.
[Part2](/blog/2013/07/18/asp-dot-net-mvc4-global-error-handling/) covered how to keep the view data set by the controller for error view and master/layout.
 In this part, we will cover how to use a custom model.

   With default exception handling mechanism of the framework, you are stuck with the model framework supplies. When an exception is caught by the framework, It creates an object of the type ```HandleErrorInfo``` 
and passes to the Error View.
 The framework doesn't provide any provision to extend this model or use different model altogether. 

We will see how to provide an hook to extend the model. 

 <!-- more -->
Let's do this with our example of displaying customer contact information based on the context. This context could vary on things such customer location, type of error, or customer serving system, tech support etc. 
Let's define a model for this.
     
``` csharp CustomModel 
    public class CustomModel
    {
        public Exception Exception { get; set; }
        public SupportInfo SupportInfo { get; set; }
    }

    public class SupportInfo
    {
       public string ContactNo{get;set;}
    }
```
Let's see how we would have done if it were happy path views. 
We would

1. create a model to encapsulate this information
2. load the model from data store or some other system based on the context 
3. set this model in the Controller/Action
4. View uses this model to display the information.

When we do above steps, the code would look as shown below.

``` csharp
    public ActionResult SomeAction() {
     //controller context
     var supportInfo = GetSupportInfo(ControllerContext);//a call to either your repo, or service
     return View(supportInfo);//which sets the model on the view
    }
``` 
  That's a clean separation of concerns, of course the beauty of this code structure is coming from MVC pattern. 

  We would expect same thing happening even for Error Views. But that's not the case. The framework handles error views differently. It sets the model and doesn't carry the ViewData set in Controller/Actions. 
 
   We have only one option to implement this. We will have to put the logic of loading the model in the error page it self. That fixes the problem, but it's not clear separation of concerns.

   What we need to solve this problem correctly is a way to the framework use our model instead of default model HandlerErrorInfo.
 if we define model for our customer support information, 
  ```SupportInfo```, 
   We need to LoadSupportInfo, and set that as a model, similar to what we would normally do for happy path views.
``` csharp
    //controller context
    var supportInfo = GetSupportInfo(ControllerContext);
    ViewResult.ViewData.Model = supportInfo;
``` 
   
It turns out that we can this by extending HandleError filter. 
   HandleError allows you specify error(default is Error.cshtml) or master/layout for your error view. Here is how you can specify different error, and layout.

``` csharp   
  [HandleError(Master="_ErrorLayout", View="ErrorWithCustomModel")]
```   
  This is a good thing that HandleError allows custom views and layouts, but not custom model.  

 We can add something to this filter to allow us to specify a custom model.
The filter should call our code that loads the data and sets the model on view.
We are essentially providing a model to the framework. So we can add a new property to filter, we can call it ModelProvider.


``` csharp   
  [HandleErrorWithCustomModel(View="ErrorWithCustomModel", ModelProvider=typeof(ErrorModelProvider))]
```

  Now we need define a contract for ModelProvider. 
As Filter has two items it can pass to this model provider: ErrorContext, and the default model it creates. Let's define the signature

``` csharp contract 
public interface IErrorModelProvider
    {
        object GetModel(ExceptionContext context, HandleErrorInfo defaultModel);
    }
```

We can implement it as shown below.
  
``` csharp implementation 
    public class ErrorModelProvider :IErrorModelProvider
    {
        public object GetModel(ExceptionContext context, HandleErrorInfo defaultModel)
        {
            return new CustomModel
                {
                   Exception = context.Exception,
                   SupportInfo = GetSupportInfo(context) 
               };
        }
        SupportInfo GetSupportInfo(ExceptionContext context){
         return new SupportInfo{ 
              ContactNo = "1800SUPPORT"; 
           };// or you can load it from external source
        }
    }
```
 Now that we have custom model, let's extend the default filter to use this model.

``` csharp 
   [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true, Inherited = true)]
    public class HandleErrorWithCustomModelAttribute : HandleErrorAttribute
    {
        public Type ModelProvider { get; set; }

        public override void OnException(ExceptionContext filterContext)
        {
            base.OnException(filterContext);
            var result = filterContext.Result as ViewResult;
            var modelProvider = Activator.CreateInstance(ModelProvider) as IErrorModelProvider;
            var model = modelProvider.GetModel(filterContext, result.Model as HandleErrorInfo);
            result.ViewData = new ViewDataDictionary(model);
            //If you need ViewData set by controller, You can add items of Controller.ViewData to View's ViewData.
        }
    }
```

Let's add this filter to Home/Index as shown below.

```  csharp HomeController.cs

  [HandleErrorWithCustomModel(View="ErrorWithCustomModel", ModelProvider = typeof(ErrorModelProvider))]
      
 public ActionResult Index()
        {
            ViewBag.Temperature = "67";
            throw new ArgumentNullException();
            ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
            return View();
  }
```

if you run the application now, you should see Custom Error View with custom model, displaying ContactNo correctly from model.

{% img /images/posts/aspdotnetmvc/error_custom_model.png %}

  In this part, we have seen how to extend the default exception filter to use custom models.
