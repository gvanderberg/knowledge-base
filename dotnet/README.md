# dotnet

## global.json overview

* [global.json overview](https://docs.microsoft.com/en-us/dotnet/core/tools/global-json)

If you don't specify `global.json` file, the latest SDK tools version on the build agent will be used.
Considering the fact that our build agents are running multiple dotnet SDK versions, it is recommended to include the `global.json` file in the root of the repo to guarantee consistent behaviour.

The following example shows how to use the latest feature band and patch version installed of a specific major and minor version. The JSON shown disallows any SDK version earlier than 3.1.102 and allows 3.1.102 or any later 3.1.xxx version, such as 3.1.103 or 3.1.200.

```json
{
  "sdk": {
    "version": "3.1.102",
    "rollForward": "latestFeature"
  }
}
```
