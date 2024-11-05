<html>
# Introduction 
This is the MEF modules for Dynamics Marketing Lead Management. It is responsible for
- Entity Scoring
- Lead Qualification.

# What is MEF
See [Module Execution Framework](https://dynamicscrm.visualstudio.com/OneCRM/_wiki/wikis/OneCRM.wiki/37382/Module-Execution-Framework)

# Getting Started

## Permissions

### MEF security group

You must join the `mefexppartners@microsoft.com` alias on https://aka.ms/idweb. Note that permissions can take up to 12 hours to fully propogate once approved.

### NuGet + Credential Provider

You probably already have `nuget.exe` and a credential provider somewhere on your `PATH`, but if not, you can download them from [here](https://dynamicscrm.pkgs.visualstudio.com/_apis/public/nuget/client/CredentialProviderBundle.zip). Just unzip somewhere and place on your `PATH`


## Prerequisites

| Syntax | Supported Version |
| ------ | ----------------- |
| Python | 3.10              |
| Spark  | 3.3.2             |
| Java   | Java 8/11         |
                    
[See MEF requirements](https://dynamicscrm.visualstudio.com/OneCRM/_wiki/wikis/OneCRM.wiki/37398/MEF-Dev-Guide?anchor=python-version-support-in-mef)

### Notes
`Python`

- By default, the installer will automatically install pip. Make sure to check the box to add Python to your `PATH` during installation, to run the pip commands conveniently.

- If you faced any issue during running make sure that you are using the specific supported python version '3.10.x'.

`Java`

For windows, run one of the following from an elevated command shell:

```
choco install zulu8
choco install zulu11
```

Alternatively, [download the Microsoft build of OpenJDK](https://learn.microsoft.com/en-us/java/openjdk/download#openjdk-11).


If you have more than Java version installed, make sure to setup `JAVA_HOME` to point to a supported version e.g.

```powershell
 $env:JAVA_HOME="C:\Program Files\Java\jdk-11"
```

`Hadoop binaries`

For spark to run locally, there are a few hadoop binaries that are required, or else writing to local storage will fail. 

To download `hadoop-3.2.2`, run:
```powershell
Install-HadoopWinutils  -PathInRepository 'hadoop-3.2.2/bin ' -DestinationPath = 'c:\hadoop-3.2.2\bin' # make sure to enlist first.
```

The scipt will update `HADOOP_HOME` and `Path` environment variables.

# Development

## Virtual environment

It is recommended to do development inside a virtual environment (just a containerized environment). You can do this via the following ways:
- your IDE (Visual Studio, pycharm, etc) 
- manually from an `elevated` PowerShell

```powershell
# Make sure to enlist first. Run inside the module where requirement.txt file is located. Both parameters are optional.
# e.g. cd C:\dev\cxp-main\src\DataProcessingModules\LeadManagement\LeadManagement.Module

Initialize-VirtualPythonEnvironment -EnvName "testenv" -Requirements requirements-dev.txt 
```

The script will create a new virtual environment, configure the required packages and activate the environment. You can create as many as you like. Virtual environments will be created under `venvs` folder.

To exit your current virtual environment:
```powershell
deactivate
```

To activate a virtual environment, either run the creation cmdlet above, or execute:
```powershell
.\venvs\{MY_TARGET_ENV}\Scripts\activate
```

## Running tests
Unit tests are written with `pytest`. You can run them either in your IDE, or locally in your virtual environment:

```powershell
# activate your venv first
.\venvs\{MY_TARGET_ENV}\Scripts\activate

# to run individual test modules
pytest .\tests\{test file}.py

# to run all tests
pytest

# to profile tests runs
pytest --profile

# alternatively, if you have more than one pytest installed
.\venvs\{MY_TARGET_ENV}\Scripts\pytest.exe

```
`pytest` is configured to discover all tests under `tests` folder by default.

### Examining stderr/stdout output
By default `pytest` will show output only for tests that failed to pass. If you need to see all output, run pytest with `-rA`. This can be specified in `pytest.ini`

```
[pytest]
addopts =-rA -p no:warnings
```

More information on available option can be found [in pytest documentation](https://docs.pytest.org/en/7.1.x/how-to/output.html)

### Enabling detailed logs
Import fixture `debug_logging_configuration` and pass it with key `LoggingConfiguration` in configs to the execute method of models. This will enable verbose logging, and print dataframe contents.

```
    configs={
        "ModelMetadata": model_metadata,
        "LoggingConfiguration": debug_logging_configuration,
        "LeadQualificationQuery": qualification_query,
    },
``` 

Same trick can be used in data processing configuration when running in TIP in real MEF.

```
  "modelScalarInputs": {
    "ModelMetadata": {
      "ModelId": "f8d4f9fe-00b3-e811-a97b-000d3a36cc30",
      "OrganizationId": "953c8495-9766-ed11-9985-000d3a1484a7",
      "RequestEpochMinutes": 28093568
    },
    "LoggingConfiguration": {
      "LogLevel": "DEBUG"     <======
    },
    "ScoringQuery": "<Q>"
  },
```

## Profiling
Virtual environments come pre-installed with `cprofilev` and `pytest-profiling`.

- Instructions for `cprofilev` can be found [here](https://github.com/ymichael/cprofilev)
- To use `pytest-profiling`, simply specify `--profile` parameter:

```powershell
pytest .\tests\{test file}.py --profile
```

## Linting
Virtual environments come pre-installed with `pylama` and `black`, which are the tools of choice.

To configure `VSCode` for `pylama`:

1. Install the Python extension in VSCode
2. Ctrl+Shft+P -> `Python: Select Interpreter` and set `.\venvs\{MY_TARGET_ENV}\Scripts\python.exe`
3. Ctrl+Shft+P -> `Python: Enable Linting` -> `on`
4. Ctrl+Shft+P -> `Python: Select Linter` -> `pylama`
5. Ctrl+Shft+P -> `Python: Run Linting`

To configure `VSCode` for `black`:
1. `Files->Preferences->Settings->Python>Formatting: Black path` and set `.\venvs\{MY_TARGET_ENV}\Scripts\black.exe` 
2. If this doesn't work, run `python -m pip install black` in your venv again.

For both tools, the executables need to be in the same directory as the interpreter (aka `python.exe`) otherwise it won't work.

## Creating a local distribution:

To test module packages locally, run the cmdlet blow. The script will build the model, and create a nuget and a wheel package. Packages won't be published to any feed, so it is safe to run.

From an elevated PowerShell, run: 
```powershell
./CreatePackage.ps1 -Version "0.0.1"
```

# Deployment

Details for MEF module deployment with all the needed pre-requisite and troubleshoots can be found [here](https://dynamicscrm.visualstudio.com/OneCRM/_wiki/wikis/OneCRM.wiki/38164/Registering-a-Module-in-Dataverse)

## TIP Deployment

To deploy changes on TIP , you need to register new module version following below steps:
- Make sure that the **CXP MEF LeadManagement Official** passed succesfully.
- Make sure that a newer version for package **Dynamics.Marketing.Models.LeadManagement** is uploaded on **cjo-feed**
- Create release [Module Registration Dev/Test/PreProd](https://dev.azure.com/powerbi/Insights%20Apps%20Platform/_release?definitionId=35&view=mine&_a=releases) with overriding the below parameters:
  - **NugetName:** Dynamics.Marketing.Models.LeadManagement
  - **NugetVersion:** Version of the package to deploy.

## PROD Deployment

To deploy changes on PROD , you need to register new module version following below steps:
- Make sure that you already done TIP deployment first (mentioned in the previous step)
- Make sure you have the needed permissions to:   
  - Submit a release you need to be in the SG : CIO_Lead_Management_Submitters_All
  - Approve the release you need to be in the SG : CIO_Lead_Management_Approvers_All
- Create release [Module Registration FRE/Prod](https://dev.azure.com/powerbi/Insights%20Apps%20Platform/_release?view=mine&_a=releases&definitionId=45) with overriding the below parameters:
  - **NugetName:** Dynamics.Marketing.Models.LeadManagement
  - **NugetVersion:** Version of the package to deploy that was already deployed on TIP.
  - **PartnerServiceTreeGuid:** 6f0fe454-35ff-42ef-aacf-f8a67b940803
</html>
