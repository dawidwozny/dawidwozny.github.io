---
title: Clasic and fluent builder
date: 2025-03-16 12:00:00 +0100
categories: [C#, Azure]
tags: [C#, patterns, builder]     # TAG names should always be lowercase
mermaid: true
---


# Introduction
Builder is one of my favourite programing pattern. It is simple, at least in it's base form, and can drastically improve readability. I have recently used it to make my integration tests more maintainable and would like to share my thoughts.

## Problem
Integration test are type of tests where you want to test some components alltoghether. When it comes to REST API this would involve sending some HTTP request. The service I was testing here was actually comand line tool and in order to test it I had to send command along with arguments and options.

If I had just a few commands without options and testing just happy cases I would probably did not bother doing anything fancy. However, things were getting messy and wanted to do something about it. Additionaly the software were in early stage and I knew will be changing interface. 

Let's consider command to be executed in pwsh:

``` powershell
gcd nipkg build `
    --content-src-dir 'testdata\nipkg\test-pkg-content' `
    --target-root-dir 'BootVolume/manual-tests/gcd-build-test' `
    --package-name 'gcd-build-test' `
    --package-version '0.5.0-1' `
    --package-destination-dir 'build-test-output-dir'
```

When translated to C# string looks:

``` csharp
var command = "gcd nipkg build --content-src-dir 'testdata\\nipkg\\test-pkg-content' --target-root-dir 'BootVolume/manual-tests/gcd-build-test' --package-name 'gcd-build-test' --package-version '0.5.0-1' --package-destination-dir 'build-test-output-dir'
```

This can be easily improved with string array:
``` csharp
var command = new  string[]
{
    "gcd",
    "nipkg",
    "build",
    "--content-src-dir", "testdata\nipkg\test-pkg-content",
    "--target-root-dir", "BootVolume/manual-tests/gcd-build-tes",
    "--package-name","cd-build-test",
    "--package-version","0.5.0-1",
    "--package-destination-dir","build-test-output-dir"
};
```

However, the problems I have got is that:
* Section `"gcd","nipkg","build"` need to be repeated for every call.
* Option names need to be memorized by programmer (me) or checked/copied every single time.
* Option names are being repeated with every call and If they were to be changed need to be chaned everywhere.

The software was in early stage and haven't decided on the interface yet. Did not want to change everything once the option name has been changed. Additionaly wanted to a test few cases with every command.

If I were building interface for command I knwo it works, I would probably use some DTO and function to call it.
``` csharp
public record BuildCommandDto(
    string ContentSrcDir,
    string TargetDir,
    string PackageName,
    string PackageVersion,
    string PackageDestinationDir);
```

But in this perticular case I actually wanted to have some flexibility to:
* send command with not supported option
* send command without required option
* send command with typo

And still:
* don't need to memorize option names
* don't repeat begining of the command 
* change option in one place

I know, it is like eating apple and having apple but it is possible with builders.

Before talking about the actuall solution let me describe builder itself. 


### Builders
There are quie a few variation of builder pattern like:
* Classic Builder
* Fluent Builder
* Fluent Builder With Recursive Generics
* Facade Builder

but here I will be describing just the basic ones:
* Classic Builder
* Fluent Builder

The initial examples I give, will not solve the actuall problem. They are actually very similar to building array. Their sole purpose is to compare Classic and Fluent Builders to each other.

### Clasic Builder
Classic builder is something I rarely use and not because it is bad. I just love the syntax Fluent Builder so much that don't want to use anything else. However, Fluent Builder can pose some chalanges when it comes to inheritance and wanted to compare it to Classic Builder in that regards.

#### Usage
``` csharp
var builder = new ClassicCommandBuilder();
builder.WithArgument("gcd");
builder.WithArgument("nipkg");
builder.WithArgument("build");
builder.WithOption("--content-src-dir","testdata\\nipkg\\test-pkg-content");
builder.WithOption("--target-root-dir","BootVolume/manual-tests/gcd-build-test");
builder.WithOption("--package-name","cd-build-test");
builder.WithOption("--package-version","0.5.0-1");
builder.WithOption("--package-destination-dir","build-test-output-dir");
var command = builder.Build();
```

#### Implementation
``` csharp
public class ClassicCommandBuilder
{
    private readonly List<string> _args = [];
    public void WithArgument(string argument)
    {
        _args.Add(argument);
    }
    public void WithOption(string name, string argument)
    {
        _args.Add(name);
        _args.Add(argument);
    }
    public string[] Build() => _args.ToArray();
}
```

#### Diagram
``` mermaid
classDiagram
    class ClassicCommandBuilder {
        - List<string> _args
        + WithArgument(string argument)
        + WithOption(string name, string argument)
        + Build() string[]
    }
```
### Fluent Builder
Fluent Builder is my first choice when it comes to builders.
Fluent interface is something which allows you to chain function and is quite easy to implement. Every function must return Builder itself instead of void.

#### Usage
```csharp
var command = new FluentCommandBuilder()
    .WithArgument("gcd")
    .WithArgument("nipkg")
    .WithArgument("build")
    .WithOption("--content-src-dir", "testdata\\nipkg\\test-pkg-content")
    .WithOption("--target-root-dir", "BootVolume/manual-tests/gcd-build-test")
    .WithOption("--wrong-name-option", "cd-build-test")
    .WithOption("--package-version", "0.5.0-1")
    .WithOption("--package-destination-dir", "build-test-output-dir")
    .Build();
```

#### Implementation
```csharp
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
## Solution
`FluentCommandBuilder` improved the interface slightly but not as much for me to choose it over `string[]` implementation. The actual solution is presented below. This approach gives me the best results:
* begining of the command is prepended automatically
* no need to remember all the option names
* option names can be changed in one place
* neat syntax
* can remove some options
* can introduce some typos in option name if necessary

> Note that I have reused 'FluentArgumentsBuilder' using composition and created methods `WithArgument`, `WithOption` that delegates work to `FluentArgumentsBuilder`

There are two reason for it:
* I tend to use 'composition over inheritance' rule whenever I can.
* Using Fluent Builder with inheritance has some chalanges.

The actual truth is that I wanted to use inheritance here. I got some commands which share the same options and this approach could save me a lot of time but I couldn't. At least in the first go.

#### Usage
```csharp
var command = new BuildCommandBuilder()
    .WithContentSrcDirOption("testdata\\nipkg\\test-pkg-content")
    .WithTargetRootDirOption("BootVolume/manual-tests/gcd-build-test")
    .WithOption("--wrong-name-option","test") 
    .WithPackageVersionOption("0.5.0-1")
    .WithPackageDestinationDirOption("build-test-output-dir")
    .Build();
```

#### Implementation
```csharp
public class BuildCommandBuilder
{
    private readonly FluentCommandBuilder _builder = new();

    public BuildCommandBuilder()
    {
        _builder.WithArgument("gcd");
        _builder.WithArgument("nipkg");
        _builder.WithArgument("build");
    }

    public BuildCommandBuilder WithContentSrcDirOption(string value)
    {
        _builder.WithOption("--content-src-dir", value);
        return this;
    }
    
    public BuildCommandBuilder WithTargetRootDirOption(string value)
    {
        _builder.WithOption("--target-root-dir", value);
        return this;
    }
    
    public BuildCommandBuilder WithPackageNameOption(string value)
    {
        _builder.WithOption("-package-name", value);
        return this;
    }
    
    public BuildCommandBuilder WithPackageVersionOption(string value)
    {
        _builder.WithOption("--package-version", value);
        return this;
    }

    public BuildCommandBuilder WithPackageDestinationDirOption(string value)
    {
        _builder.WithOption("--package-destination-dir", value);
        return this;
    }
    
    public BuildCommandBuilder WithArgument(string value)
    {
        _builder.WithArgument(value);
        return this;
    }
    
    public BuildCommandBuilder WithOption(string name, string value)
    {
        _builder.WithOption(name, value);
        return this;
    }
    public string[] Build() =>
        _builder.Build();
}
```
### Fluent Builder with intheritance
So what is the fuss about using fluent builder with inheritance?

Let's consider implementation below.

Let's assume that I have defined `BuildCmdArgumentBuilderUsingInheritance` using inheritance from `FluentArgumentBuilder` the code become a bit simpler:
```csharp
public class BuildCommandBuilderUsingInheritance : FluentCommandBuilder
{
    public BuildCommandBuilderUsingInheritance()
    {
        WithArgument("gcd");
        WithArgument("nipkg");
        WithArgument("build");
    }

    public BuildCommandBuilderUsingInheritance WithContentSrcDirOption(string value)
    {
        WithOption("--content-src-dir", value);
        return this;
    }
    
    public BuildCommandBuilderUsingInheritance WithTargetRootDirOption(string value)
    {
        WithOption("--target-root-dir", value);
        return this;
    }
    
    public BuildCommandBuilderUsingInheritance WithPackageNameOption(string value)
    {
        WithOption("-package-name", value);
        return this;
    }
    
    public BuildCommandBuilderUsingInheritance WithPackageVersionOption(string value)
    {
        WithOption("--package-version", value);
        return this;
    }

    public BuildCommandBuilderUsingInheritance WithPackageDestinationDirOption(string value)
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

