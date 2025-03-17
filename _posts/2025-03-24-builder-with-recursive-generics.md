---
title: Builder With Recursive Generics Pattern
date: 2025-03-16 12:00:00 +0100
categories: [C#, Patterns]
tags: [C#, patterns, builder]     # TAG names should always be lowercase
mermaid: true
---

# Intro

In my previous post [Builder Pattern](https://dwozny.com/posts/builder-pattern/) I shared my experience with using **Fluent Builder** pattern and explain it's limitations. 

### Problem with Builder
At the core of **Fluent Builder** is the fact that it returns itself. As soon as you use method of parent class, it will return parent builder and you won't have access to child class methods anymore. This problem does not exist with **Classic Builder** since you don't depend on what the method returns and mutate builder's data instead. On the other hand you could use **Fluent Builder** the same way **Clasic Builder** is used but then what's the point creating it this way in the first place.

### Recursive Generics
I will start with fancy names and try to explain it later.

Recursive Generics is also know as *

The Curiously Recurring Template Pattern (CRTP) is a design pattern where a class is derived from a generic base class that takes the derived class itself as a type parameter.


AI generated:
``` csharp
// Generic base class that takes the derived type as a type parameter
public class Base<TSelf> where TSelf : Base<TSelf>
{
    public TSelf DoSomething()
    {
        Console.WriteLine("Doing something...");
        return (TSelf)this;  // Return derived type
    }
}

// Derived class that passes itself as the generic argument
public class Derived : Base<Derived>
{
    public Derived AnotherAction()
    {
        Console.WriteLine("Another action in Derived class.");
        return this;  // Returns correct type
    }
}

// Usage
class Program
{
    static void Main()
    {
        var obj = new Derived()
            .DoSomething()  // Comes from Base<TSelf>
            .AnotherAction();  // Comes from Derived

        Console.WriteLine("Execution finished.");
    }
}
```

####

Using the example from previous post 

``` csharp
public abstract class GenericCommandBuilder<TSelf> where TSelf : GenericCommandBuilder<TSelf>
{
    private readonly List<string> _args = [];
    public TSelf WithArgument(string argument)
    {
        _args.Add(argument);
        return (TSelf)this;
    }
    public TSelf WithOption(string name, string argument)
    {
        _args.Add(name);
        _args.Add(argument);
        return (TSelf)this;
    }
    public string[] Build() => _args.ToArray();
}
```


``` csharp
public class FluentCommandBuilder
{
    private readonly List<string> _args = [];
    public FluentCommandBuilder WithArgument(string argument)
    {
        _args.Add(argument);
        return this;
    }
    public FluentCommandBuilder WithOption(string name, string argument)
    {
        _args.Add(name);
        _args.Add(argument);
        return this;
    }
    public string[] Build() => _args.ToArray();
}
```

The implementation 
``` csharp
public class BuildCommandBuilderUsingGenerics : GenericCommandBuilder<BuildCommandBuilderUsingGenerics>
{
    public BuildCommandBuilderUsingGenerics()
    {
        WithArgument("gcd");
        WithArgument("nipkg");
        WithArgument("build");
    }

    public BuildCommandBuilderUsingGenerics WithContentSrcDirOption(string value)
    {
        WithOption("--content-src-dir", value);
        return this;
    }
    
    public BuildCommandBuilderUsingGenerics WithTargetRootDirOption(string value)
    {
        WithOption("--target-root-dir", value);
        return this;
    }
    
    public BuildCommandBuilderUsingGenerics WithPackageNameOption(string value)
    {
        WithOption("-package-name", value);
        return this;
    }
    
    public BuildCommandBuilderUsingGenerics WithPackageVersionOption(string value)
    {
        WithOption("--package-version", value);
        return this;
    }

    public BuildCommandBuilderUsingGenerics WithPackageDestinationDirOption(string value)
    {
        WithOption("--package-destination-dir", value);
        return this;
    }
}
```

``` csharp
var commandBuiltByInheritacneFriendlyBuilder = new BuildCommandUsingGenerics()
    .WithContentSrcDirOption("testdata\\nipkg\\test-pkg-content")
    .WithTargetRootDirOption("BootVolume/manual-tests/gcd-build-test")
    .WithOption("--wrong-name-option","test") 
    .WithPackageVersionOption("0.5.0-1")
    .WithPackageDestinationDirOption("build-test-output-dir")
    .Build();
```

### Elephant
There is an elephant in the room. Have you noticed?
`BuildCommandBuilderUsingGenerics` is no longer generic and once you inherit from it you will have exatly the same situation we had using simple FluentBuilder from previous post.

I must say that this perticular example was not the best. First of all the comand got word in it 'build' which is harder to read. An

How about if you would like an instance of `GenericCommandBuilder`?

You can't simply do, it is generic type. You need to specify a type. 
``` csharp
var builder = new GenericCommandBuilder()
```

This also won't work 
``` csharp
var builder = new GenericCommandBuilder<GenericCommandBuilder>()
```

Neither this
``` csharp
var builder = new GenericCommandBuilder<GenericCommandBuilder<GenericCommandBuilder>>()
```

How about you would like to inherit from BuildCommandBuilderUsingGenerics because another command use the same options? There will be the same problem as with original FluentBuilder.


## More fun
More fun start if you want to keep inheritance. Let's assume that you want to keep your builder open for someone to inherit from it. 

Let's consider three levels of inheritance `Parent` <- `Child` <-


So what you can do about it?
You can keep your abstract 
