# Concourse resource for sonar scanning a dotnet core application

Provides support for [SonarQube](https://www.sonarqube.org/) scanning a C# .NET core 2 application.

## Usage

Source parameters:
* host_url - the URL to the SonarQube server (e.g. http://localhost:9000)
* login - the SonarQube login token

Parameters:
* project_name - the name of the project to show in SonarQube
* project_key - the configured project key in SonarQube
* source - the directory containing the source code to analyze
* version (optional) - reference to the project version file - as generated by the semver resource
* projects (optional) - the directories that contain the source to scan (defaults to all directories containing a csproj file)
* extra_params (optional) - list of key=value pairs that will be passed to sonarqube; this can be used to pass additional reports to sonar

N.B. the SonarQube MSBuild process is executed in the context of the source directory, so any extra_params that reference files need to be relative to this location (e.g. ../coverage-reports/opencovercoverage.xml).

The following ```get``` results in the return of the quality gates status of the project in a file called ```sonar-qualitygates-status.json``` in the following [format](https://next.sonarqube.com/sonarqube/web_api/api/qualitygates/project_status).

## Requirements

As part of the scan, a ```dotnet restore``` and ```dotnet build``` will be executed against each directory listed in the ```projects``` parameter (or all directories containing a ```.csproj``` file).  Any Visual Studio solution file (.sln) will not be used.  As such, SonarQube requires that each ```.csproj``` file contains a ```ProjectGuid``` project property.

e.g.

```
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <ProjectGuid>{2AF61F67-D12F-4E53-92BC-C0DC3EA5DAC5}</ProjectGuid>

    ...
  </PropertyGroup>
```

### Example
```
resource_types:
- name: sonarqube-scanner
  type: docker-image
  source:
    repository: subnova/concourse-dotnet-sonarscanner-resource
    tag: "latest"

resources:
- name: sonarqube-scan
  type: sonarqube-scanner
  source:
    host_url: ((sonarqube_host_url))
    login: ((sonarqube_login_token))

jobs:
- name: sonarqube
  plan:
  - get: version
  - get: my-source
  - put: sonarqube-scan
    params:
      project_name: MyProjectName
      project_key: myproject
      version: version/version
      source: my-source
      projects: MyApp MyApp.Model MyApp.Test
      extra_params:
      - sonar.cs.vstest.reportsPaths=../unit-test-results/*.trx
      - sonar.verbose=true
```
