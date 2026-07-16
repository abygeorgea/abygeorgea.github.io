---
layout: post
title: "Reading Configuration Values in Dotnet core using Options Pattern"
slug: "reading-configuration-values-in-dotnet-core-using-ioptions-pattern"
date: 2020-12-17T13:26:34+11:00
comments: true
categories:
  - Dotnet Core
keywords:
  - Dotnet Core
  - Configuration
  - IOPtions
description: "How to read configuration Values in dotnet core using Options Pattern"
---


Normally every project will have some values to be read from config files like user name, connection strings, environment specific details etc. 

## Dotnet Framework ##
In dotnet framework projects, this can be easily done by using Configuration Manager. This is available when we add System.Configuration assembly reference.

Let us look at an example of a app config file like below

``` xml

<?xml version="1.0" encoding="utf-8" ?>  
<configuration>  
  <startup>  
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />  
  </startup>  
  <appSettings>  
    <add key="Url" value="https://www.saucedemo.com/"/>  
    <add key="UserName" value="standard_user"/>  
    <add key="Password" value="secret_sauce"/>  
  </appSettings>  
</configuration> 

```

Storing user name and password in app config is not a best practise. For sake of simplicity , let us keep it here. In dotnet framework , we can read above config values with below

``` CSharp
var Url = ConfigurationManager.AppSettings["Url"];  
var UserName = ConfigurationManager.AppSettings["UserName"]; 
var Password = ConfigurationManager.AppSettings["Password"]; 

```

##Dotnet Core##

Now let us look how to read above config values in a dotnet core project .

Let us look at a appsettings.json file 

``` javascript
{
  "SauceDemoDetails": {
    "Url": "https://www.saucedemo.com/",
    "UserName": "standard_user",
    "Password": "secret_sauce"
  },
  "MyKey":  "My appsettings.json Value",
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

There are different approaches to read the config file. Let us look at couple of them.

##IConfiguration##

One of the approach to consume this is using IConfiguration Interface. We can inject this to the class constructor where we need to consume these config values.

``` CSharp
//This is not entire class file. It just show the main areas where we need to make changes.

//Using statement
using Microsoft.Extensions.Configuration;

// Make sure to create a private variable and inject IConfiguration to constructor

IConfiguration _configuration;
public demo(IConfiguration configuration)
{
    _configuration = configuration;
}


//Usage is as below. Use GetValue method and specify the type. 
//Also we need to give entire hierarchy staring with section value to the keyname separated by :

var Url = _configuration.GetValue<string>("SauceDemoDetails:Url");  
var UserName = _configuration.GetValue<string>("SauceDemoDetails:UserName");  
var Password = _configuration.GetValue<string>("SauceDemoDetails:Password");  
```

However injecting IConfiguration is not a good practise since we are not sure what configuration is this class now depend on. Also class should know the entire hierarchy of configuration making it tightly coupled.


##IOptions##

Another way to access these config values is by using Options Pattern . Documentation of that can be found [here](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-5.0). The options pattern uses classes to provide strongly typed access to groups of related settings. It also provides way to validate details.

In this pattern, we need to create an options class corresponding to the config value

``` CSharp
public class SauceDemoDetailsOptions
{
    public const string SauceDemoDetails = "SauceDemoDetails";

    public string Url { get; set; }
    public string UserName { get; set; }
    public string Password { get; set; }
}

```

According to documentation, options class should be non abstract class with public parameterless constructor. All public get - set properties of the type are bound and fields are not bound. Hence in above class, Url, UserName, Password are bound to config values and SauceDemoDetails are not bound.

Let us see how we can bind the configuration file to above created class. In startup.cs file, do below

``` CSharp

// Use configuration builder to load the appsettings.json file

public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true);

    if (env.IsDevelopment())
    {
        builder.AddUserSecrets();
    }

    builder.AddEnvironmentVariables();
    Configuration = builder.Build();
}

// Configure Services to bind
//Need to give entire hierarchy separated by : . In below example, I use the constant string from the class to specify so that it doesn't need to be hard coded here.
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<SauceDemoDetailsOptions>(Configuration.GetSection(SauceDemoDetailsOptions. SauceDemoDetails));
}

```



Inorder to use this, we need to inject IOptions to the consuming class.

``` CSharp

//This is not entire class file. It just show the main areas where we need to make changes.

//Using statement
using Microsoft.Extensions.Options;

// Make sure to create a private variable and inject IOptions to constructor. IOptions expose Value property which contains the details of object.

private SauceDemoDetailsOptions _ sauceDemoDetailsOptions;
public demo(IOptions<SauceDemoDetailsOptions> sauceDemoDetailsOptions)
{
    _sauceDemoDetailsOptions = sauceDemoDetailsOptions.Value;
}

//Usage is as below.  :

var Url = _sauceDemoDetailsOptions.Url;  
var UserName = _sauceDemoDetailsOptions.UserName;  
var Password = _sauceDemoDetailsOptions.Password; 

```


One important point to remember is Ioptions interface doesn't support reading configuration data after app has started. Hence any changes to appsettings.json after the app has started will not be effective till next restart.  If we need to recompute the values every-time, then we should use *IOptionsSnapshot* interface. This is a scoped interface which cannot be injected to Singleton service. The usage of this is same as IOptions interface. We also need to make sure that configuration source also supports reload on change.


##Validations##

Let us look into how to implement validations into this. In above examples, if there are any mistakes like typo ,missing fields etc, then those fields will not be bound. However it will not error out there and will continue execution , till it throws an exception where it require these missing fields.  We can add validations from DataAnnotations library to identify these earlier. 

DataAnnotations provide various validator attributes like Required, Range, RegularExpression, String Length etc.

Let us add [Required] to all properties as below.

``` CSharp

using System.ComponentModel.DataAnnotations;
public class SauceDemoDetailsOptions
{
    public const string SauceDemoDetails = "SauceDemoDetails";
	[Required]
    public string Url { get; set; }
    [Required]
    public string UserName { get; set; }
    [Required]
    public string Password { get; set; }
}

```

We will also have to change the *startup* class to do validations after we bind.

``` CSharp
public void ConfigureServices(IServiceCollection services)
{
 
services.AddOptions<SauceDemoDetailsOptions>()
		  .Bind(Configuration.GetSection(SauceDemoDetailsOptions. SauceDemoDetails)
		  .ValidateDataAnnotations();
		  

}

```