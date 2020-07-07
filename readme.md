# `NuGet.Config` Environment Variable Sample

This sample demonstrates how to use environment variables in your `NuGet.Config` to reduce risk of accidentally sharing secrets.

## Set up `NuGet.Config`

First, create a `nuget.config` file. One way is to start with the template from the dotnet cli by running `dotnet new nugetconfig` (note the lack of `.` in the `nuget.config` filename). Alternatively, you can copy a complete sample (for example this repo) and paste in your text editor.

You can then use [`nuget.exe sources add`](https://docs.microsoft.com/nuget/reference/cli-reference/cli-ref-sources) or [`dotnet nuget add source`](https://docs.microsoft.com/dotnet/core/tools/dotnet-nuget-add-source) to modify the `nuget.config` file, saving your password in clear text (using `-StorePasswordInClearText` or `--store-password-in-clear-text`). You probably need to use the `-ConfigFile` or `--configfile` arguments to ensure that NuGet writes to your `nuget.config` file, unless [NuGet/Home#7867](https://github.com/NuGet/Home/issues/7867) is resolved. Alternatively, you can manually edit the file in your text editor.

Finally, edit your `nuget.config` in a text editor, and change the value for your feed's username and password values to the names of the environment variables you wish to use.

Here is sample `nuget.config` file, with a source named `PrivateFeed`, URL was replaced with `url`, and the username environment variable name is `NuGetFeedUserName` and the password environment variable name is `NuGetFeedPAT`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="PrivateFeed" value="url" />
  </packageSources>
  <packageSourceCredentials>
    <PrivateFeed>
      <add key="Username" value="%NuGetFeedUserName%" />
      <add key="ClearTextPassword" value="%NuGetFeedPAT%" />
    </PrivateFeed>
  </packageSourceCredentials>
</configuration>
```

## Set up Azure Pipeline YAML

Different CI systems will have different details on how to do this, but using Azure Pipelines as an example, the first step is to define the secrets for the pipeline. I will use a variable group, as it allows the variables to be shared across multiple pipelines/build/release definitions.

First, create a variable group and define your variables, marking the variables as secret.

![Image showing variable group with two variables, `NuGetFeedUserName` and `NuGetFeedPAT`](docs/variable_group.png)

Next, you can either edit your build definition to set the build to use the variable group, or you can use [the YAML syntax to import the variable group](https://docs.microsoft.com/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=example%2Cparameter-schema#variables):

```yaml
variables:
- group: string # name of a variable group
```

Finally, Azure DevOps will keep secret variables secret, unless the specific step or task unmasks it using [the `env` syntax](https://docs.microsoft.com/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#secret-variables).

Also note that `env` didn't work in my own testing when using the `DotNetCoreCLI` task, so I just invoke the dotnet CLI directly.

```yaml
- script: dotnet restore -v:d
  displayName: Restore
  env:
    NuGetFeedUserName: $(NuGetFeedUserName)
    NuGetFeedPAT: $(NuGetFeedPAT)
```
