# Restore error when two projects in same directory

To reproduce the bug take the following steps:

```cmd
> dotnet restore 
  Determining projects to restore...
  Restored R:\example\example.csproj (in 5.7 sec).
```

Notice how `example.csproj` was restored here but `other.csproj` was not. While they share
the same `project.assets.json` (by design or coincidence) the `.nuget.g.props` and`
`.nuget.g.targets` are only restored for `example.csproj`. This means the `<PackageReference>`
in `other.csproj` is not properly setup and that causes an error on the next step.

```cmd
> dotnet build --no-restore
MSBuild version 17.3.2+561848881 for .NET
  example -> R:\example\bin\Debug\net6.0\example.dll
CSC : error CS1617: Invalid option '11' for /langversion. Use '/langversion:?' to list supported values. [R:\example\other.csproj]

Build FAILED.

CSC : error CS1617: Invalid option '11' for /langversion. Use '/langversion:?' to list supported values. [R:\example\other.csproj]
    0 Warning(s)
    1 Error(s)

Time Elapsed 00:00:00.48
```

Now this can be fixed by manually restoring `other.cpsroj`

```cmd
R:\ 
> dotnet restore .\example\other.csproj
  Determining projects to restore...
  Restored R:\example\other.csproj (in 125 ms).
R:\ 
> dotnet build --no-restore
MSBuild version 17.3.2+561848881 for .NET
  other -> R:\example\bin\Debug\net6.0\other.dll
  example -> R:\example\bin\Debug\net6.0\example.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:00.53
```

This strongly implies that `dotnet restore` is not fully executing when the projects are 
in the same directory.


