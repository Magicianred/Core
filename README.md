![Datasilk Logo](http://www.markentingh.com/projects/datasilk/logo.png)

# Datasilk Core MVC
#### An MVC Framework for ASP.NET Core
Datasilk Core is an ultra-fast, light-weight alternative to ASP.NET Core MVC 5 that supports HTML scaffolding and simple web services.

Instead of managing a complex ASP.NET Core web application and all of its configuration, simply include this framework within your own ASP.NET Core Web Application project, follow the installation instructions below, and start building your website!

## Installation

1. Include the Nuget Package `Datasilk.Core.Mvc` within your ASP.NET Core project.

That's it! Next, learn how to use the Datasilk Core MVC framework to build web Controllers & web services.

## Startup.cs

Make sure to include the middelware within your `Startup` class `Configure` method.

```
public virtual void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	app.UseDatasilkMvc(new MvcOptions()
	{
		IgnoreRequestBodySize = true,
		WriteDebugInfoToConsole = true,
		Routes = new Routes()
	});
}
```

## Page Requests

All page request URLs are mapped to controllers that inherit the `Datasilk.Core.Web.Controller` class. For example, the URL `http://localhost:7770/products` would map to the class `MyProject.Controllers.Products`. Each controller contains one method, `Render`, which is used to serve a web page to the user's web browser.

**/Views/Home/home.html**
```
<div class="hero">
	<h1>{{title}}</h1>
	<h3>{{description}}</h3>
</div>
```

**/Controllers/Home.cs**
```
namespace MyProject.Controllers
{
    public class Home: Datasilk.Core.Web.Controller
    {
        public override string Render(string body = "")
		{
			//render page
			var view = new View("/Views/Home/home.html");
			view["title"] = "Welcome";
			view["description"] = "I like to write software";
			AddScript("/js/views/home/home.js");
			return view.Render();		
		}
	}
}
```

In the example above, a user tries to access the URL `http://localhost:7770/`, which (by default) will render the contents of the `MyProject.Controllers.Home` class. This class loads `/Views/Home/home.html` into a `View` object and replaces the `{{title}}` & `{{description}}` variables located within the `home.html` file with the text, "Welcome!" & "I like to write software". Then, the page returns the contents of `view.Render()`.

#### Controller Hierarchy
You could create controllers that inherit other controllers, then `return base.Rennder(view.Render())` to create a cascade. This would be useful to have a base controller that renders the header & footer of a web page, while all other controllers render the body.

**/Views/Shared/layout.html**
```
<html>
    <head>...</head>
    <body>{{content}}</body>
</html>
```

**/Controllers/Layout.cs**
```
namespace MyProject.Controllers
{
    public class Home: Datasilk.Core.Web.Controller
    {
        public override string Render(string body = "")
		{
			//render layout
			var view = new View("/Views/Shared/layout.html");
			view["content"] = body;
			return view.Render();
		}
	}
}
```

**/Controllers/Home.cs**
```
namespace MyProject.Controllers
{
    public class Home: Layout
    {
        public override string Render(string body = "")
		{
			//render page
			var view = new View("/Views/Home/home.html");
			return base.Render(view.Render());		
		}
	}
}
```

In the example above, the `Home` controller inherits from the `Layout` controller class, then calls `base.Render`, which will allow the `Layout` controller to inject the home page HTML content into the body of the HTML layout.

> NOTE: `MyProject.Controllers.Home` is the default class that is instantiated if the URL contains a domain name with no path structure. 

### Controller Routing
To render web Controllers based on complex URL paths, the MVC framework relies heavily on the first part of the request path to determine which class to instantiate. For example, if the user accesses the URL `http://localhost:7770/blog/2019/11/09/Progress-Report`, Datasilk Core MVC initializes the `MyProject.Controllers.Blog` class. 

The request path is split up into an array and passed into the `PathParts` field within the `Datasilk.Core.Web.Controller` class. The `PathParts` array is used to determine what type of content to load for the user. If we're loading a blog post like the above example, we can check the `PathParts` array to find year, month, and day, followed by the title of the blog post, and determine which blog post to load.

```
namespace MyProject.Controllers
{
    public class Blog: Datasilk.Core.Web.Controller
    {
        public override string Render(string body = "")
		{
			if(PathParts.length > 1){
				//render blog entry
				var date = new Date(PathParts[1] + "/" + PathParts[2] + "/" + PathParts[3]);
				var title = PathParts[3] || "";
				var view = new View("/Views/Blog/entry.html");
				//get blog entry from database using date & title
				return view.Render();
			}
			else
			{
				//render blog home page
				var view = new View("/Views/Blog/blog.html");
				return view.Render();
			}           
		}
	}
```

### Access Denied
If your web page is protected behind security and must display an `Access Denied` page, you can use: 

```return AccessDenied<IController>()```

 from within your `Datasilk.Core.Web.Controller` class `Render` method, which will render a controller of your choosing with a response status code of `403`.

## Web Services
The Datasilk Core MVC framework comes with the ability to call *RESTful* web APIs. All web API calls are executed from `Datasilk.Core.Web.Service` classes.

#### Example

```
namespace MyProject.Services
{
    public class User: Datasilk.Core.Web.Service
    {
		[POST]
		public string Authenticate(string email, string password)
		{
			//authenticate user
			//...
			if(authenticated){
				return Success();
			}else{
				S.Response.StatusCode = 500;
				return "";
			}
		}
	}
}
```

In the example above, the user would send an `AJAX` `POST` via JavaScript to the Uri `/api/User/Authenticate` to authenticate their email & password. The data sent to the server would be formatted using the JavaScript function, `JSON.stringify({email:myemail, password:mypass})`, and the data properties would be mapped to C# method arguments.

### Web Service Response
All `Datasilk.Core.Web.Service` methods should return a string. You can easily return a JSON object as well.

```
using System.Text.Json;
...
return JsonSerializer.Serialize(obj);
```

## Routes.cs
If you would like to map requests to controllers & services directly, create a new class that inherits `Datasilk.Core.Web.Routes`. For example:

```
using Microsoft.AspNetCore.Http;
using Datasilk.Core.Web;

public class Routes : Datasilk.Core.Web.Routes
{
    public override IController FromControllerRoutes(HttpContext context, Parameters parameters, string name)
    {
        switch (name)
        {
            case "": case "home": return new Controllers.Home();
            case "login": return new Controllers.Login();
            case "dashboard": return new Controllers.Dashboard();
        }
        return null;
    }

    public override IService FromServiceRoutes(HttpContext context, Parameters parameters, string name)
    {
        switch (name)
        {
            case "": case "user": return new Services.User();
            case "dashboard": return new Services.Dashboard();
        }
        return null;
    }
}
```

#### Why Routing?
By routing new class instances using the `new` keyword, you bypass the last resort for Datasilk Core MVC, which is to create an instance of your `Controller` or `Service` class using `Activator.CreateInstance`, taking 10 times the amount of CPU ticks to instatiate. You don't have to use routing, but it does speed up performance slightly.


## Optional: Datasilk Core Javascript Library
Learn more about the optional Javascript library, [Datasilk/CoreJs](https://github.com/Datasilk/CoreJs), which contains various client-side features used to build a modern web application. The library includes features such as message alert boxes, popup modals, drag & drop functionality, HTML templating, and custom scrollbars.