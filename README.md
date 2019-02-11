# FsLibLog

FsLibLog is a single file you can copy paste or add through [Paket Github dependencies](https://fsprojects.github.io/Paket/github-dependencies.html) to provide your F# library with a logging abstraction.  This is a port of the C# [LibLog](https://github.com/damianh/LibLog).


## Getting started

### 1. Put the file into your project

#### Option 1

Copy/paste [FsLibLog.fs](https://github.com/TheAngryByrd/FsLibLog/blob/master/src/FsLibLog/FsLibLog.fs) into your library 

#### Option 2

Read over [Paket Github dependencies](https://fsprojects.github.io/Paket/github-dependencies.html).

Add the following line to your `paket.depedencies` file.

```
github TheAngryByrd/FsLibLog src/FsLibLog/FsLibLog.fs
```

Then add the following line to projects with `paket.references` file you want FsLibLog to be available to.

```
File: FsLibLog.fs
```


### 2. Replace its namespace with yours

To alleviate potential naming conflicts, it's best to replace FsLibLog namespace with your own.

Here is an example with FAKE 5:

```fsharp
Target.create "Replace" <| fun _ ->
  Shell.ReplaceInFiles
    [ "FsLibLog", "MyLib.Logging" ]
    (!! "paket-files/TheAngryByrd/FsLibLog/src/FsLibLog/FsLibLog.fs")
```


## Using in your library

### Open namespaces 

```fsharp
open FsLibLog
open FsLibLog.Types
```

### Get a logger

There are currently three ways to get a logger.

- `getCurrentLogger` - Creates a logger. It's name is based on the current StackFrame.
- `getLoggerFor` - Creates a logger given a `'a` type.
- `getLogger` - Creates a logger given a `Type`.


### Set the loglevel, message, exception and parameters

Choose a LogLevel. (Fatal|Error|Warn|Info|Debug|Trace).

There are helper methods on the logger instance, such as `logger.warn`.

These helper functions take a `(Log -> Log)` which allows you to amend the log record easily with functions in the `Log` module.  You can use function composition to set the fields much easier.

```fsharp
logger.warn(
    Log.setMessage "{name} Was said hello to"
    >> Log.addParameter name
)
```

The set of functions to augment the `Log` record are

- `Log.setMessage` - Amends a `Log` with a message
- `Log.setMessageThunk` - Amends a `Log` with a message thunk.  Useful for "expensive" string construction scenarios.
- `Log.addParameter` - Amends a `Log` with a parameter.
- `Log.addParameters` - Amends a `Log` with a list of parameters.
- `Log.addException` - Amends a `Log` with an exception.



### Full Example:

```fsharp
namespace SomeLib
open FsLibLog
open FsLibLog.Types


module Say =
    let logger = LogProvider.getCurrentLogger()

    let hello name  =
        logger.warn(
            Log.setMessage "{name} Was said hello to"
            >> Log.addParameter name
        )
        sprintf "hello %s." name

    let fail name =
        try
            failwithf "Sorry %s isnt valid" name
        with e ->
            logger.error(
                Log.setMessage "{name} was rejected."
                >> Log.addParameter name
                >> Log.addException  e
            )
```


## Currently supported providers

- [Serilog](https://github.com/serilog/serilog)

---

## Builds

MacOS/Linux | Windows
--- | ---
[![Travis Badge](https://travis-ci.org/TheAngryByrd/FsLibLog.svg?branch=master)](https://travis-ci.org/TheAngryByrd/FsLibLog) | [![Build status](https://ci.appveyor.com/api/projects/status/github/TheAngryByrd/fsliblog?svg=true)](https://ci.appveyor.com/project/TheAngryByrd/FsLibLog)
[![Build History](https://buildstats.info/travisci/chart/TheAngryByrd/FsLibLog)](https://travis-ci.org/TheAngryByrd/FsLibLog/builds) | [![Build History](https://buildstats.info/appveyor/chart/TheAngryByrd/FsLibLog)](https://ci.appveyor.com/project/TheAngryByrd/fsliblog)  


---

### Building


Make sure the following **requirements** are installed in your system:

* [dotnet SDK](https://www.microsoft.com/net/download/core) 2.0 or higher
* [Mono](http://www.mono-project.com/) if you're on Linux or macOS.

```
> build.cmd // on windows
$ ./build.sh  // on unix
```

#### Environment Variables

* `CONFIGURATION` will set the [configuration](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-build?tabs=netcore2x#options) of the dotnet commands.  If not set it will default to Release.
  * `CONFIGURATION=Debug ./build.sh` will result in things like `dotnet build -c Debug`
* `GITHUB_TOKEN` will be used to upload release notes and nuget packages to github.
  * Be sure to set this before releasing

### Watch Tests

The `WatchTests` target will use [dotnet-watch](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/dotnet-watch.md) to watch for changes in your lib or tests and re-run your tests on all `TargetFrameworks`

```
./build.sh WatchTests
```

### Releasing
* [Start a git repo with a remote](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)

```
git add .
git commit -m "Scaffold"
git remote add origin origin https://github.com/user/MyCoolNewLib.git
git push -u origin master
```

* [Add your nuget API key to paket](https://fsprojects.github.io/Paket/paket-config.html#Adding-a-NuGet-API-key)

```
paket config add-token "https://www.nuget.org" 4003d786-cc37-4004-bfdf-c4f3e8ef9b3a
```

* [Create a GitHub OAuth Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
    * You can then set the `GITHUB_TOKEN` to upload release notes and artifacts to github
    * Otherwise it will fallback to username/password


* Then update the `RELEASE_NOTES.md` with a new version, date, and release notes [ReleaseNotesHelper](https://fsharp.github.io/FAKE/apidocs/fake-releasenoteshelper.html)

```
#### 0.2.0 - 2017-04-20
* FEATURE: Does cool stuff!
* BUGFIX: Fixes that silly oversight
```

* You can then use the `Release` target.  This will:
    * make a commit bumping the version:  `Bump version to 0.2.0` and add the release notes to the commit
    * publish the package to nuget
    * push a git tag

```
./build.sh Release
```


### Code formatting

To format code run the following target

```
./build.sh FormatCode
```

This uses [Fantomas](https://github.com/fsprojects/fantomas) to do code formatting.  Please report code formatting bugs to that repository.
