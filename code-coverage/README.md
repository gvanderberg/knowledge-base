## Code coverage

```bash
dotnet tool install -g coverlet.console
coverlet ./bin/Debug/../*.UnitTests.dll --target "dotnet" --targetargs "test --no-build"
coverlet ./bin/Debug/../*.UnitTests.dll --target "dotnet" --targetargs "test --no-build" --exclude "[*]{namespace}*"
```

install coverlet.collector nuget package

```bash
dotnet test --collect:"XPlat Code Coverage"
```

```bash
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"path to coverage.cobertura.xml" -targetdir:"coverageresults" -reporttypes:Html
```
