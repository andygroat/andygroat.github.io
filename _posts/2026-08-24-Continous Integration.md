---
title: "Continous Integration"
date: 2026-08-24
---

## Repos

[Classic N-Tier Repo Link](https://github.com/andygroat/todo-classic-api)

[VSA Repo Link](https://github.com/andygroat/todo-vsa-api)

## Intro

In the previous posts we have setup the APIs with some basic functionality and then added some testing to ensure that there are no bugs and we don't break that functionality with future updates. Now we need to have a look at the repos and ensuring that when code is being checked that it can still build and the code is high enough quality. This is where CI (Continous Integration) comes in.

The complete yml pipelines can be found in the repos.

Continous Integration is usually accompanied with Continous Deployment (CD) so you get a full CI/CD workflow. The CI part is quite large, so I will focus on it in this post and hopeully get round to doing another post on CD later on.

## Continous Integration (CI)

**Continous Integration** is a practice that requires frequently committing code to a repository, which will show errors early and reduces the amount of code to debug to fix the error.

After each code push or pull request we will want to run a **CI Pipeline** which will ensure that the code in the repo will build and tests will run.

For dotnet we can have a very simple CI pipeline such as this:

```yml
name: Basic-CI

on:
  push:
    branches:
      - main

env:
  DotNetVersion: 10.x
  Solution: demo.slnx

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Setup DotNet
        uses: actions/setup-dotnet@v6
        with:
          dotnet-version: ${{ env.DotNetVersion }}

      - name: Restore Dependencies
        run: dotnet restore ${{ env.Solution }}

      - name: Build
        run: dotnet build ${{ env.Solution }} --configuration Release --no-restore

      - name: Test
        run: dotnet test ${{ env.Solution }} --configuration Release --no-build

```

This will do a nice job of a basic build of the solution and run of the tests, however we need something a bit more advanced where we also want to check the code coverage levels and publish the build on completion.

Also worth noting that when we hit dotnet 10 and start to use the new Microsoft Testing Platform (MTP), there are some changes required to the testing step of the pipeline, so lets build up the pipeline.

### Name & Trigger

The name of the pipeline and how the pipeline will be triggered should start of the pipeline.

```yml
# Name the Workflow/Pipeline, this appears on the Github Actions page, so keep it easy to understand and recognisable.
name: main-continous-integration

# Setup the triggers for the pipeline, this the event that occurs that trigger this pipeline to run.
# Trigger the CI pipeline on push or pull request (main branch)
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

### Variables

Then declare the variables that will be used throughtout the pipeline.

```yml
# Declare environment variables for use throughout the pipeline
env:
  DotNetVersion: 10.x
  Solution: Todo.Classic.API/Todo.Classic.API.slnx
  WebProject: Todo.Classic.API/Todo.Classic.API/Todo.Classic.API.csproj
```

### Permissions

Setup permissions that the pipeline will need.  This is required because we will want the pipeline to write results of the tests and code coverage to the pull requests.  If the permissions are not set and the pipeline attempts to write comments to a PR, the pipeline will fail.

```yml
permissions:
  # Setup the permissions for the pipeline, this is required to allow the pipeline to run and access the repo.
  # The permissions are set to read-only for the contents of the repo, and read/write for the pull requests.
  # This is required to allow the pipeline to add comments to pull requests.
  contents: read
  checks: write
  pull-requests: write
```

### Jobs

The basics of the pipeline configuration has been setup with the steps above, so now we can get on with configuring the job itself. Since we are focusing on CI, this pipeline will only have one job, and that is for the build itself. We also want to specify the server/agent that the pipeline will run on.

```yml
# Setup the job steps. This defines the steps that the pipeline will follow
jobs:
  build:
    # Specify the server/agent that the pipeline will run on
    runs-on: ubuntu-latest
```

#### Steps

Now we will move to configure the steps for the build job. These are configured within the steps section.

```yml
# Configure the steps for the pipeline
steps:
```

#### Checkout

The first step will be to checkout the code from the repo so the agent has the code for building.

```yml
# Check the latest code out of the repo
- name: Checkout
  uses: actions/checkout@v6
```

#### Setup DotNet

Ensure that the correct version of dotnet is setup on the agent.

```yml
# Setup the DotNet environment on the agent
- name: Setup DotNet
  uses: actions/setup-dotnet@v6
  with:
    dotnet-version: ${{ env.DotNetVersion }}
```

#### Restore Dependencies

Restore the dependencies (nuget packages and any other packages used) for the solution.

```yml
# Restore the dependencies for the project
- name: Restore Dependencies
  run: dotnet restore ${{ env.Solution }}
```

#### Build Solution

Build the solution, here you can specify the configuration for the build, and adding in the `--no-restore` since we have already done the restore of the packages in the previous step.

```yml
# Build the solution
- name: Build
  run: dotnet build ${{ env.Solution }} --configuration Release --no-restore
```

#### Test

Next up is the testing step. The below is confgiuration for running the tests.  It uses the same configuration as the build step, and we specify `--no-build` since we ran the build previous step. We then specify the `--collect` switch which is used to gather code coverage statistics. Another key switch in there is `--settings`, which points to the `.runsettings` file which resides at the same level as the solution. This file contains the settings for code coverage to ensure it will ignore generated code and anything else we don't want included in code coverage statistics. This file is the same as Visual Studio will auto detect and use when reporting code coverage.

```yml
# Run the tests and get the code coverage
- name: Test
  run: dotnet test ${{ env.Solution }} --configuration Release --no-build --verbosity normal --collect:"XPlat Code Coverage" --results-directory ./coverage --logger trx --settings Todo.Classic.API/.runsettings
```

The above works nicely for the older testing libraries that rely on VSTest, however for the newer libraries such as TUnit, this uses the new Microsoft Testing Platform (MTP), so the switches passed the `dotnet test` command changes.

```yml
# Run the tests and get the code coverage
- name: Test
  run: dotnet test --solution ${{ env.Solution }} --configuration Release --no-build --verbosity normal -- --coverage --coverage-output-format cobertura --coverage-settings codecoverage.config.xml --results-directory ./coverage --report-trx
```

Here after the standard switches for configuration and not building, there is an extra `--`, and its this switch that tells the CLI that MTP is being used and then we specify the switches for the code coverage. The settings for code coverage in MTP is specified in `codecoverage.config.xml` file which is in the root of the repo.

Along with a change to the switches, there are also some updates in the solution that need to checked. The `.csproj` file for the test projects need to have `<TestingPlatformDotnetTestSupport>true</TestingPlatformDotnetTestSupport>` set in the `PropertyGroup`. And one update that was a gotcha for me was that in the root of the repo a `global.json` file is also required with following contents.

```xml
{
    "test": {
        "runner": "Microsoft.Testing.Platform"
    }
}
```

Check out the [MS Test](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test) documentation for more information on the differences and switches.

#### Test & Code Coverage Results

After the test have run successful, we want to do something with the results of the tests and code coverage.

First up build a report with the test results.

```yml
# Publish the test results to Github Actions
- name: Publish test results
  uses: dorny/test-reporter@v3
  if: always()
  with:
    name: .NET Test Results
    path: '**/*.trx'
    reporter: dotnet-trx
```

Next build a report for the code coverage. There are multiple reports generated for code coverage, so combine the reports into a single report. Note the settings for the Code Coverage Report, in there you can specify if the pipeline fail if the coverage is below threshold levels, and you can specify the threshold levels.

```yml
# Combine the coverage reports into one report
- name: Combine Coverage Reports
  uses: danielpalme/ReportGenerator-GitHub-Action@5.5.11
  with:
    reports: "**/*.cobertura.xml"
    targetdir: "${{ github.workspace }}"
    reporttypes: "HtmlInline;Cobertura"
    verbosity: "Info"
    title: "Code Coverage"
    tag: "${{ github.run_number }}_${{ github.run_id }}"
    customSettings: ""
    toolpath: "reportgeneratortool"

# Upload the combined coverage report as an artifact
- name: Upload Combined Coverage XML
  uses: actions/upload-artifact@v7
  with:
    name: coverage
    path: ${{ github.workspace }}/Cobertura.xml
    retention-days: 5

# Create the code coverage report
- name: Code Coverage Report
  uses: irongut/CodeCoverageSummary@v1.3.0
  with:
    filename: Cobertura.xml
    badge: true
    fail_below_min: true
    format: markdown
    hide_branch_rate: false
    hide_complexity: false
    indicators: true
    output: both
    thresholds: '60 80'
```

If the pipeline was run on PR, add a comment to the PR with the code coverage.

```yml
# If the pipeline was trigger by a PR, add a comment with the coverage
- name: Add Coverage PR Comment
  uses: marocchino/sticky-pull-request-comment@v2
  if: github.event_name == 'pull_request'
  with:
    recreate: true
    path: code-coverage-results.md
```

#### Publish

The final step of of the pipeline is to publish the built solution and store it in the artifacts so it can be used later on.

```yml
# Publish the project to a folder for deployment
- name: Publish
  run: dotnet publish ${{ env.WebProject }} -c Release --property:PublishDir='./todo-momo-api'

# Upload the published project as an artifact for deployment
- name: Upload artifact for deployment job
  uses: actions/upload-artifact@v7
  with:
    name: .net-app
    path: ./Todo.Vsa.Api/Todo.Vsa.Api/bin/Release/net10.0
```

## Summary

That is all of the steps need to create a CI pipeline that will build the solution, run the tests and report on the tests, the code coverage of those tests and publish the solution for deployment later on either manually or as part of CD pipeline.

Happy Coding!
