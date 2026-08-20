# .NET Quick Cheatsheet
_A simple beginner guide_

---

## What this cheatsheet covers

This quick cheatsheet explains:

- what `.NET` is
- how to check if it is installed
- how to start a project
- how to run a project
- how to build a project
- how to create common project types
- basic folder and file ideas
- common `dotnet` CLI commands

---

# 1. What is .NET?

`.NET` is the platform and toolchain used to build and run C# programs.

You commonly use it for:

- console apps
- web apps
- APIs
- desktop apps
- libraries

Think of it like this:

- **C#** = the language
- **.NET** = the platform + tools that compile and run the code

---

# 2. Check if .NET is installed

Open your terminal and run:

```bash
dotnet --version
```

If installed, it prints a version number.

Example:

```text
8.0.100
```

You can also see more info:

```bash
dotnet --info
```

---

# 3. How to start a new project

The most common beginner project is a **console app**.

Create one like this:

```bash
dotnet new console -n MyApp
```

### Breakdown

- `dotnet` = run the .NET CLI
- `new` = make a new project
- `console` = use the console app template
- `-n MyApp` = name the project `MyApp`

This creates a new folder named `MyApp`.

---

# 4. Move into the project folder

```bash
cd MyApp
```

You usually want to do this before running or editing the project.

---

# 5. Run the project

Inside the project folder:

```bash
dotnet run
```

This builds and runs the app.

For a fresh console template, you should see something like:

```text
Hello, World!
```

---

# 6. Basic beginner workflow

A very common `.NET` beginner workflow is:

```bash
dotnet new console -n MyApp
cd MyApp
dotnet run
```

Then edit the code and run again:

```bash
dotnet run
```

That is the main beginner loop.

---

# 7. Build the project without running it

```bash
dotnet build
```

This compiles the project.

Use this when you want to check for errors without launching the program.

---

# 8. Clean build files

```bash
dotnet clean
```

This removes build output files.

Useful when something gets weird and you want a cleaner rebuild.

---

# 9. Restore dependencies

```bash
dotnet restore
```

This restores packages and dependencies the project needs.

Often `dotnet build` and `dotnet run` do this automatically when needed, but the command still exists and is useful.

---

# 10. Common project files

When you create a console project, you often get files like:

```text
MyApp/
├── MyApp.csproj
├── Program.cs
├── obj/
└── bin/
```

## Meaning

### `Program.cs`
Usually contains your main code.

### `MyApp.csproj`
Project file that tells .NET how the project is configured.

### `bin/`
Build output.

### `obj/`
Temporary build/intermediate files.

---

# 11. What is the `.csproj` file?

Example:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

## Important pieces

### `OutputType`
`Exe` means this project builds into an executable app.

### `TargetFramework`
Example:

```xml
<TargetFramework>net8.0</TargetFramework>
```

This says which .NET version/framework the project targets.

### `ImplicitUsings`
Lets C# include some common namespaces automatically.

### `Nullable`
Turns nullable reference type analysis on.

---

# 12. Edit the main code

A basic `Program.cs` might look like this:

```csharp
Console.WriteLine("Hello, world!");
```

Change it to something else:

```csharp
Console.WriteLine("Hello from my .NET app!");
```

Then run:

```bash
dotnet run
```

---

# 13. Create other project types

## Class library

```bash
dotnet new classlib -n MyLibrary
```

Use this when you want reusable code instead of a standalone app.

## Web app / API

```bash
dotnet new webapi -n MyApi
```

This creates a starter web API project.

## Empty web app

```bash
dotnet new web -n MyWebApp
```

## Solution file

```bash
dotnet new sln -n MySolution
```

A solution can hold multiple projects.

---

# 14. Add a project to a solution

First create a solution:

```bash
dotnet new sln -n MySolution
```

Then add a project:

```bash
dotnet sln MySolution.sln add MyApp/MyApp.csproj
```

This is common in larger multi-project setups.

---

# 15. Create a project in the current folder

Sometimes you do not want a new subfolder.

Use:

```bash
dotnet new console
```

That creates the project in the current folder.

If needed, make a folder first:

```bash
mkdir MyApp
cd MyApp
dotnet new console
```

---

# 16. List available templates

```bash
dotnet new list
```

This shows available project templates.

Older examples online may use:

```bash
dotnet new --list
```

Both are related to listing templates, depending on SDK version habits and docs.

---

# 17. Add a NuGet package

If your project needs an external package:

```bash
dotnet add package PackageName
```

Example:

```bash
dotnet add package Newtonsoft.Json
```

This adds the package reference to your project.

---

# 18. Remove a NuGet package

```bash
dotnet remove package PackageName
```

Example:

```bash
dotnet remove package Newtonsoft.Json
```

---

# 19. List package references

```bash
dotnet list package
```

Useful to see what packages a project uses.

---

# 20. Run a specific project

If you are in a bigger folder with multiple projects, you can do:

```bash
dotnet run --project MyApp/MyApp.csproj
```

This tells .NET exactly which project to run.

---

# 21. Build Release mode

```bash
dotnet build -c Release
```

## Meaning

- `-c` = configuration
- `Release` = optimized release build

Common configs:

- `Debug`
- `Release`

For everyday learning, `Debug` is normal.  
For a more final build, `Release` is common.

---

# 22. Publish a project

Publishing creates output for distribution.

```bash
dotnet publish -c Release
```

This produces publish output you can deploy or move elsewhere.

---

# 23. Run a built DLL manually

Sometimes after building, you may run the output with:

```bash
dotnet MyApp.dll
```

This is common if the build produced a framework-dependent app.

---

# 24. Create and run a class library plus console app setup

Very common structure:

- one console app
- one library project

Example:

```bash
dotnet new sln -n DemoSolution
dotnet new console -n DemoApp
dotnet new classlib -n DemoLibrary
dotnet sln DemoSolution.sln add DemoApp/DemoApp.csproj
dotnet sln DemoSolution.sln add DemoLibrary/DemoLibrary.csproj
dotnet add DemoApp/DemoApp.csproj reference DemoLibrary/DemoLibrary.csproj
```

Now the app can use code from the library.

---

# 25. Add a project reference

To make one project use another project:

```bash
dotnet add MyApp/MyApp.csproj reference MyLibrary/MyLibrary.csproj
```

This is different from adding a NuGet package.

- **package** = external dependency
- **project reference** = another local project in your solution/folder

---

# 26. Typical beginner project structure

A small beginner console app often looks like:

```text
MyApp/
├── MyApp.csproj
├── Program.cs
├── Helpers.cs
├── Models/
│   └── Player.cs
├── bin/
└── obj/
```

This is not mandatory, but it is very normal.

---

# 27. Common beginner mistakes

## Mistake 1: Running commands outside the project folder

If you do:

```bash
dotnet run
```

in the wrong folder, .NET may not find your project.

Fix:
- `cd` into the correct folder
- or use `--project`

---

## Mistake 2: Editing `bin/` or `obj/`

Usually do **not** edit files inside:

- `bin/`
- `obj/`

Those are build output/intermediate folders.

Edit your source files like:

- `Program.cs`
- `*.cs`
- `*.csproj`

---

## Mistake 3: Confusing C# with .NET

Remember:

- C# = language
- .NET = platform/tooling/runtime

They are connected, but not the same thing.

---

## Mistake 4: Forgetting to restore/build after changes to packages

If dependencies change, run:

```bash
dotnet restore
dotnet build
```

or just try:

```bash
dotnet run
```

which often handles restore/build automatically.

---

# 28. Super common commands recap

## Check version

```bash
dotnet --version
```

## Create console project

```bash
dotnet new console -n MyApp
```

## Enter project folder

```bash
cd MyApp
```

## Run

```bash
dotnet run
```

## Build

```bash
dotnet build
```

## Clean

```bash
dotnet clean
```

## Restore

```bash
dotnet restore
```

## Create solution

```bash
dotnet new sln -n MySolution
```

## Add package

```bash
dotnet add package PackageName
```

## Add project reference

```bash
dotnet add MyApp/MyApp.csproj reference MyLibrary/MyLibrary.csproj
```

## Publish

```bash
dotnet publish -c Release
```

---

# 29. Fast mental model

Here is the quick mental picture:

## Start a project
```bash
dotnet new console -n MyApp
```

## Enter it
```bash
cd MyApp
```

## Run it
```bash
dotnet run
```

## Edit code
Change `Program.cs`

## Run again
```bash
dotnet run
```

That is the basic .NET beginner cycle.

---

# 30. One sentence memory hook

**Use `dotnet new` to create, `dotnet run` to test, `dotnet build` to compile, and `dotnet publish` to prepare an app for distribution.**
