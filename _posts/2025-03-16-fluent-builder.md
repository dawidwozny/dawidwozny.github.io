---
title: Clasic and fluent builder
date: 2025-03-16 12:00:00 +0100
categories: [C#, Azure]
tags: [C#, patterns, builder]     # TAG names should always be lowercase
---


# Intro
Builder is one of my favourite programing pattern. It is simple, at least in it's base form, and very useful. It's indispensable in making test neat and maintainable. I started to write integration test for a command line tool. As you can imagine in comand line everything is esentailly string. When command is simple it is rather straightforward task to write, however, when you start using options and you have quite a few of them, also when you want to simulate variouse cases then things starting to get a bit messy. Using builder seemd to be perfect match to provide strongly(ish) typed interface.

``` powershell
gcd nipkg build `
    --content-src-dir 'testdata\nipkg\test-pkg-content' `
    --target-root-dir 'BootVolume/manual-tests/gcd-build-test' `
    --package-name 'gcd-build-test' `
    --package-version '0.5.0-1' `
    --package-destination-dir 'build-test-output-dir'
```

If you would like to transform it into C# code this would look like
``` csharp
var arguments = "gcd nipkg build --content-src-dir 'testdata\nipkg\test-pkg-content' --target-root-dir 'BootVolume/manual-tests/gcd-build-test' --package-name 'gcd-build-test' --package-version '0.5.0-1' --package-destination-dir 'build-test-output-dir'
```

And yes you can make some tweaks to make it better but I am not a fan of manipulating strings


### Builders

### Clasic Builder
Classic builder is something I rarely use but it's good to mention it in the begining

``` csharp
public class ClassicArgumentBuilder
{
    private readonly List<string> _args = [];
    public void AddArgument(string argument)
    {
        _args.Add(argument);
    }
    public void AddOption(string name, string argument)
    {
        _args.Add(argument);
    }
    public string[] Build() => _args.ToArray();
}
```
It's usage

``` csharp
var builder = new ClassicArgumentBuilder();
builder.AddArgument("gcd");
builder.AddArgument("nipkg");
builder.AddArgument("build");
builder.AddOption("--content-src-dir","testdata\nipkg\test-pkg-content");
builder.AddOption("--target-root-dir","BootVolume/manual-tests/gcd-build-test");
builder.AddOption("--package-name","cd-build-test");
builder.AddOption("--package-version","0.5.0-1");
builder.AddOption("--package-destination-dir","build-test-output-dir");

var aruments = builder.Build();
```

### Fluent Builder
Fluent build is something I use very often and I am big fan of fluent interface.
Fluent interface is something which allows you to chain function and write them in a very neat way.

```csharp
public class FluentArgumentBuilder
{
    private readonly List<string> _args = [];
    public FluentArgumentBuilder AddArgument(string argument)
    {
        _args.Add(argument);
        return this;
    }
    public FluentArgumentBuilder AddOption(string name, string argument)
    {
        _args.Add(argument);
        return this;
    }
    public string[] Build() => _args.ToArray();
}
```
Usage
```csharp
var aruments = new FluentArgumentBuilder()
    .AddArgument("gcd")
    .AddArgument("nipkg")
    .AddArgument("build")
    .AddOption("--content-src-dir", "testdata\nipkg\test-pkg-content")
    .AddOption("--target-root-dir", "BootVolume/manual-tests/gcd-build-test")
    .AddOption("--package-name", "cd-build-test")
    .AddOption("--package-version", "0.5.0-1")
    .AddOption("--package-destination-dir", "build-test-output-dir")
    .Build();
```
## Next step
Using fluent interface allready some improvements but still had to repeat a lot an remember all the options.

So I wrote specific builder for the specific command.

```csharp
var arguments = new BuildCmdArgBuilder()
    .WithContentSrcDirOption("testdata\nipkg\test-pkg-content")
    .WithTargetRootDirOption("BootVolume/manual-tests/gcd-build-test")
    .WithPackageNameOption("cd-build-test")
    .WithPackageVersionOption("0.5.0-1")
    .WithPackageDestinationDirOption("build-test-output-dir")
    .Build();
```

```csharp
public class BuildCmdArgBuilder
{
    private readonly FluentArgumentBuilder _builder = new();

    public BuildCmdArgBuilder()
    {
        _builder.AddArgument("gcd");
        _builder.AddArgument("nipkg");
        _builder.AddArgument("build");
    }

    public BuildCmdArgBuilder WithContentSrcDirOption(string value)
    {
        _builder.AddOption("--content-src-dir", value);
        return this;
    }
    
    public BuildCmdArgBuilder WithTargetRootDirOption(string value)
    {
        _builder.AddOption("--target-root-dir", value);
        return this;
    }
    
    public BuildCmdArgBuilder WithPackageNameOption(string value)
    {
        _builder.AddOption("-package-name", value);
        return this;
    }
    
    public BuildCmdArgBuilder WithPackageVersionOption(string value)
    {
        _builder.AddOption("--package-version", value);
        return this;
    }

    public BuildCmdArgBuilder WithPackageDestinationDirOption(string value)
    {
        _builder.AddOption("--package-destination-dir", value);
        return this;
    }
}
```

Note that I have reused previous 'FluentArgumentsBuilder' using composition. There are two reason for it:
* I tend to use 'composition over inheritance' rule whenever I can. Although in this perticular case I knew that I will have many options with the same name for other commands and actually wanted to use.
* Using Fluent Builder with is chalanging

### Chalange of using Fluent Builder with intheritance

Let's assume that I have defined `BuildCmdArgumentBuilderUsingInheritance` using inheritance from `FluentArgumentBuilder` the code become a bit simpler:
```csharp
public class BuildCmdArgumentBuilderUsingInheritance : FluentArgumentBuilder
{
    public BuildCmdArgumentBuilderUsingInheritance()
    {
        WithArgument("gcd");
        WithArgument("nipkg");
        WithArgument("build");
    }

    public BuildCmdArgumentBuilderUsingInheritance WithContentSrcDirOption(string value)
    {
        WithOption("--content-src-dir", value);
        return this;
    }
    
    public BuildCmdArgumentBuilderUsingInheritance WithTargetRootDirOption(string value)
    {
        WithOption("--target-root-dir", value);
        return this;
    }
    
    public BuildCmdArgumentBuilderUsingInheritance WithPackageNameOption(string value)
    {
        WithOption("-package-name", value);
        return this;
    }
    
    public BuildCmdArgumentBuilderUsingInheritance WithPackageVersionOption(string value)
    {
        WithOption("--package-version", value);
        return this;
    }

    public BuildCmdArgumentBuilderUsingInheritance WithPackageDestinationDirOption(string value)
    {
        WithOption("--package-destination-dir", value);
        return this;
    }
}
```

The main problem with using Fluent Builder with inheritance is that:
**parent function will return parent builder**. Meaning that as soon as I call `WithOption("","")` which is defined in `FluentArgumentBuilder` I will no longer has access to methods defined on `BuildCmdArgumentBuilderUsingInheritance`

``` csharp
var args4 = new BuildCmdArgumentBuilderUsingInheritance()
    .WithContentSrcDirOption("testdata\nipkg\test-pkg-content")
    .WithOption("--fake-option","") // option that does not belong to cmd interface and should be handled gracefully
    .WithTargetRootDirOption("BootVolume/manual-tests/gcd-build-test")
    .WithPackageNameOption("cd-build-test")
    .WithPackageVersionOption("0.5.0-1")
    .WithPackageDestinationDirOption("build-test-output-dir")
    .Build();
```

You would not have this problem if classic builder was used but that's not what I wanted.

The solution to this chalange is `fluent builder with recursive generics` and I will be covering that in future posts.

