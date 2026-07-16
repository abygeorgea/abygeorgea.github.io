---
layout: post
title: "Dynamically create instance of a type on run time using Reflection in C#"
slug: "creating-classes-on-run-time-using-reflection-in-c-number"
date: 2021-07-31T08:36:05+10:00
comments: true
categories: []
keywords: CSharp
description: Dynamically create instance of a type on run time using Reflection
---

Reflection is the process of describing the metadata of types, methods and fields in a code. It helps to get information about loaded assemblies and elements within it like classes, methods etc. According to microsoft documentation, Reflection provides objects (of type Type) that describe assemblies, modules, and types. You can use reflection to dynamically create an instance of a type, bind the type to an existing object, or get the type from an existing object and invoke its methods or access its fields and properties. If you are using attributes in your code, reflection enables you to access them.

While creating test automation framework, we will come across scenarios where we need to create instance of an objects on run time, need to examine and instantiate types in an assembly, access attributes etc. One of the common usecase is when we create a generic framework, which will allow users to specify class name in feature files and handle it without doing any further modification. Let us look at how we can achieve those.


##Examples of Reflection##

###How to get Type of an object###

``` csharp
// Using GetType to obtain type information:
string i = "Hello World";
Type type = i.GetType();
Console.WriteLine(type); 
```

This will print *System.String*


###How to get Details loaded assembly###

``` csharp
// Using Reflection to get information of an Assembly:
Assembly info = typeof(string).Assembly;
Console.WriteLine(info);
```
This will print *System.Private.CoreLib, Version=4.0.0.0, Culture=neutral*

###How to create instance of a Class###

Creating an instance of inbuilt class can be done like below.
```Csharp
// create instance of class DateTime
DateTime dateTime =dqqw(DateTime)Activator.CreateInstance(typeof(DateTime));
```


Creating an instance of custom class is a multistep process

```Csharp
// 1. Load the dll having the class
Assembly testAssembly = Assembly.LoadFile(@"PathToDll\Test.dll");
// get type of class from just loaded assembly
Type classType = testAssembly.GetType("Test.CustomClass");

// create instance of class
object classInstance = Activator.CreateInstance(classType);

```



Activator.CreateInstance has detailed explanation of various constructors  [here](https://docs.microsoft.com/en-us/dotnet/api/system.activator.createinstance?view=net-5.0)





###How to call a method from created instance###
```Csharp
// fullNameOfClass is the complete name of the class including all namespaces. It is also assumed that , it is in same assembly, there is a default constructor and method doesn't need any parameters.

 var classHandle = Activator.CreateInstance(null, fullNameOFClass, true, 0, null, null, null, null);
  var classInstance = (className)classHandle.Unwrap();
  
 // Unwrap in above activate an object in another AppDomain, retrieve a proxy to it with the Unwrap method, and use the proxy to access the remote object. More details [here](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.remoting.objecthandle.unwrap?view=net-5.0)
 //If it is in same Appdomain, we will not need to cast
   Type t = classInstance.GetType();

//Invoking Method without parameters. If parameters are needed, pass them as an object array
   MethodInfo method = t.GetMethod(methodName);
   method.Invoke(p, null);
   
  //Getting Field value
  FieldInfo field = t.GetField(fieldName);
   return field.GetValue(p);
  
```

Seeing this in an example will help to make our understanding clear. 

1. Create SimpleCalculatorClass as below 

```Csharp
namespace SimpleConsoleApp.ReflectionExample
{
    public class Calculator
    {
        public Calculator()
        {

        }

        public double Add(double FirstValue, double SecondValue)
        {
            return FirstValue + SecondValue;
        }
    }
}

```

2. Use reflection to create an instance of above calculator class


We will start with creating a class Handle by passing full name of teh class. Then we get details of Add method. Then we call Add method by passing parameters as an object array. It will return the value as defined in the class

```Csharp
 static void Main(string[] args)
        {
            var classHandle = Activator.CreateInstance(null, "SimpleConsoleApp.ReflectionExample.Calculator");
            var calculatorObjectCreated = classHandle.Unwrap();
            Type t = calculatorObjectCreated.GetType();

            MethodInfo method = t.GetMethod("Add");
            var result =  method.Invoke(calculatorObjectCreated,new object[] { 2,5});
            Console.ReadLine();
        }
    }
```


   
   

            
