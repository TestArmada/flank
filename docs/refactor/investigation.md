# Flank investigation

This document contains investigation of flank architecture, layers and package structure.

## Motivation

Currently, the flank code is not well documented, 
and the only source of true is the implementation. 
The code base has grown over the last year, 
and it's almost impossible to keep in mind whole project.
So to make further work more doable,
it's necessary to identify hidden constraints in code,
expose them in documentation and do refactor.
This document is and entity point for further improvements.

## Layers

### Presentation

### Issues
1. `ftl.Main` command shouldn't be bound with `main` function with constructor
1. Some commands that can run domain code are doing too much.
1. Some of composing commands seem to be in wrong package.

### CLI commands

Flank commands tree.

```
ftl/cli/
├── AuthCommand.kt /
├── FirebaseCommand.kt /
├── auth
│   └── LoginCommand.kt ? !
└── firebase
    ├── CancelCommand.kt ? !
    ├── RefreshCommand.kt ? !
    ├── TestCommand.kt /
    └── test
        ├── AndroidCommand.kt /
        ├── CommandUtil.kt
        ├── CommonRunCommand.kt
        ├── IPBlocksCommand.kt /
        ├── IosCommand.kt /
        ├── NetworkProfilesCommand.kt /
        ├── ProvidedSoftwareCommand.kt /
        ├── android
        │   ├── AndroidDoctorCommand.kt ? !
        │   ├── AndroidRunCommand.kt ? !
        │   ├── AndroidTestEnvironmentCommand.kt ? !
        │   ├── configuration
        │   │   ├── AndroidLocalesCommand.kt / ^
        │   │   ├── AndroidLocalesDescribeCommand.kt ? !
        │   │   └── AndroidLocalesListCommand.kt ? !
        │   ├── models
        │   │   ├── AndroidModelDescribeCommand.kt ? !
        │   │   ├── AndroidModelsCommand.kt / ^
        │   │   └── AndroidModelsListCommand.kt ? !
        │   ├── orientations
        │   │   ├── AndroidOrientationsCommand.kt / ^
        │   │   └── AndroidOrientationsListCommand.kt ? !
        │   └── versions
        │       ├── AndroidVersionsCommand.kt / ^
        │       ├── AndroidVersionsDescribeCommand.kt ? !
        │       └── AndroidVersionsListCommand.kt ? !
        ├── ios
        │   ├── IosDoctorCommand.kt ? !
        │   ├── IosRunCommand.kt ? !
        │   ├── IosTestEnvironmentCommand.kt ? !
        │   ├── configuration
        │   │   ├── IosLocalesCommand.kt / ^
        │   │   ├── IosLocalesDescribeCommand.kt ? !
        │   │   └── IosLocalesListCommand.kt ? !
        │   ├── models
        │   │   ├── IosModelDescribeCommand.kt ? !
        │   │   ├── IosModelsCommand.kt / ^
        │   │   └── IosModelsListCommand.kt ? !
        │   ├── orientations
        │   │   ├── IosOrientationsCommand.kt / ^
        │   │   └── IosOrientationsListCommand.kt ? !
        │   └── versions
        │       ├── IosVersionsCommand.kt / ^
        │       ├── IosVersionsDescribeCommand.kt ? !
        │       └── IosVersionsListCommand.kt ? !
        ├── ipblocks
        │   └── IPBlocksListCommand.kt ?
        ├── networkprofiles
        │   ├── NetworkProfilesDescribeCommand.kt ?
        │   └── NetworkProfilesListCommand.kt ?
        └── providedsoftware
            └── ProvidedSoftwareListCommand.kt ?
```
command that:
* `?` is running domain code
* `/` routes to subcommands
* `!` is doing too much in run function
* `^` should be moved up in package hierarchy

### Domain

#### Issues

1. The core domain api is not easy to identify.
1. The domain layer of flank is not clearly separated of CLI and external APIs.
1. Domain logic is huge and complicated but there is lack of diagram for visualize it.

### Data [ consider better name ]

#### Issues

* External APIs are not hidden behind interfaces
* External API wrappers / adapters are not clearly separated from other layers.

#### External API libs

* `com.google.testing:firebase_apis:test_api`
* `com.google.api-client:google-api-client`
* `com.google.auth:google-auth-library-oauth2-http`
* `com.google.cloud:google-cloud-nio`
* `com.google.cloud:google-cloud-storage`
* `com.google.apis:google-api-services-toolresults`
* `io.sentry:sentry`
* `com.mixpanel:mixpanel-java`

#### List of external API usages in files

Based on [google_api_usecases](../gcloud/google_api_usecases.md)

1. com.google.api.client.http
    * GoogleApiLogger.kt
1. ftl.android
    * AndroidCatalog.kt
1. ftl.args
    * ArgsHelper.kt
1. ftl.config
    * Credentials.kt
    * FtlConstants.kt
1. ftl.environment
    * ListIPBlocks.kt $
    * ListLocales.kt $
    * LocalesDescription.kt $
    * NetworkProfileDescription.kt $
1. ftl.environment.android
    * AndroidModelDescription.kt $
    * AndroidSoftwareVersionDescription.kt $
    * ListAndroidDevices.kt $
    * ListAndroidSofwareVersions.kt $
1. ftl.environment.common
    * ListNetworkConfiguration.kt $
    * ListOrientations.kt $
    * ListProvidedSoftware.kt $
1. ftl.environment.ios
    * IosModelDescription.kt $
    * IosSoftwareVersionDescription.kt $
    * ListIOsDevices.kt $
    * ListIOsSofwareVersions.kt $
1. ftl.gc
    * GcAndroidDevice.kt $
    * GcAndroidTestMatrix.kt
    * GcIosMatrix.kt $
    * GcIosTestMatrix.kt
    * GcStorage.kt
    * GcTesting.kt $
    * GcTestMatrix.kt
    * GcToolResults.kt
    * UserAuth.kt
    * ftl/gc/Utils.kt $
1. ftl.gc.android
    * CreateAndroidInstrumentationTest.kt $
    * CreateAndroidLoopTest.kt $
    * CreateAndroidRobotTest.kt $
    * SetupAndroidTest.kt $
    * SetupEnvironmentVariables.kt $
    * ftl/gc/android/Utils.kt $
1. ftl.gc.ios
    * SetupIosTest.kt $
1. ftl.http
    * ExecuteWithRetry.kt
    * HttpTimeoutIncrease.kt
1. ftl.ios
    * IosCatalog.kt $
1. ftl.json
    * MatrixMap.kt $
    * OutcomeDetailsFormatter.kt $
    * SavedMatrix.kt $
1. ftl.log
    * Loggers.kt
1. ftl.mock
    * MockServer.kt
1. ftl.reports
    * HtmlErrorReport.kt
1. ftl.reports.api
    * CreateJUnitTestCase.kt $
    * CreateJUnitTestResult.kt $
    * CreateTestExecutionData.kt
    * CreateTestSuiteOverviewData.kt $
    * PerformanceMetrics.kt
    * PrepareForJUnitResult.kt
    * ProcessFromApi.kt $
    * ftl/reports/api/Utils.kt
1. ftl.reports.api.data
    * TestExecutionData.kt $
    * TestSuiteOverviewData.kt $
1. ftl.reports.outcome
    * BillableMinutes.kt $
    * CreateMatrixOutcomeSummary.kt $
    * CreateTestSuiteOverviewData.kt $
    * TestOutcome.kt $
    * TestOutcomeContext.kt $
    * ftl/reports/outcome/Util.kt $
1. ftl.reports.util
    * ReportManager.kt $
1. ftl.run
    * RefreshLastRun.kt $
1. ftl.run.common
    * FetchArtifacts.kt
    * PollMatrices.kt
1. ftl.run.platform
    * RunAndroidTests.kt $
1. ftl.run.platform.common
    * AfterRunTests.kt $
1. ftl.run.status
    * ExecutionStatusListPrinter.kt $
    * TestMatrixStatusPrinter.kt $
1. ftl.util
    * MatrixState.kt $
    * TestMatrixExtension.kt $

where
* `$` - only operates on API structures, not call methods directly

#### Google API use cases

* `auth/LoginCommand.kt`
   * Authorizing google account using OAuth2 for getting credentials.
  
* `firebase/CancelCommand.kt`
   * Sending cancel request using projectId and testMatrixId

* `firebase/RefreshCommand.kt` ?
   * Getting current test matrices' status for updating matrix file if needed
   * Polling the matrices' status for live logging until all matrices are not finished.
   * Downloading test artifacts from bucket
   
* `firebase/test/providedsoftware/ProvidedSoftwareListCommand.kt`
   * Getting softwareCatalog from testEnvironmentCatalog

* `firebase/test/ipblocks/IPBlocksListCommand.kt`
   * Getting orchestratorVersion from testEnvironmentCatalog
   
* `firebase/test/networkprofiles/NetworkProfilesDescribeCommand.kt`
   * Getting configurations from testEnvironmentCatalog through networkConfigurationCatalog
   
* `firebase/test/networkprofiles/NetworkProfilesListCommand.kt`
   * Getting configurations from testEnvironmentCatalog through networkConfigurationCatalog

* `firebase/test/android/configuration/AndroidLocalesDescribeCommand.kt`
   * Getting android device catalog for obtain locales

* `firebase/test/android/configuration/AndroidLocalesListCommand.kt`
   * Getting android device catalog for obtain locales

* `firebase/test/android/models/AndroidModelDescribeCommand.kt`
   * Getting android device catalog for obtain models

* `firebase/test/android/models/AndroidModelsListCommand.kt`
   * Getting android device catalog for obtain models

* `firebase/test/android/orientations/AndroidOrientationsListCommand.kt`
   * Getting android device catalog for obtain orientations

* `firebase/test/android/versions/AndroidVersionsDescribeCommand.kt`
   * Getting android device catalog for obtain versions

* `firebase/test/android/versions/AndroidVersionsListCommand.kt`
   * Getting android device catalog for obtain versions  

* `firebase/test/android/AndroidTestEnvironmentCommand.kt`
   * `AndroidModelsListCommand`
   * `AndroidVersionsListCommand`
   * `AndroidLocalesListCommand`
   * `ProvidedSoftwareListCommand`
   * `NetworkProfilesListCommand`
   * `AndroidOrientationsListCommand`
   * `IPBlocksListCommand`

* `firebase/test/ios/configuration/IosLocalesDescribeCommand.kt`
   * Getting ios device catalog for obtain locales
   
* `firebase/test/ios/configuration/IosLocalesListCommand.kt`
   * Getting ios device catalog for obtain locales
   
* `firebase/test/ios/models/IosModelDescribeCommand.kt`
   * Getting ios device catalog for obtain models
   
* `firebase/test/ios/models/IosModelsListCommand.kt`
   * Getting ios device catalog for obtain models
   
* `firebase/test/ios/orientations/IosOrientationsListCommand.kt`
   * Getting ios device catalog for obtain orientations
   
* `firebase/test/ios/versions/IosVersionsDescribeCommand.kt`
   * Getting ios device catalog for obtain versions
   
* `firebase/test/ios/versions/IosVersionsListCommand.kt`
   * Getting ios device catalog for obtain versions
   
* `firebase/test/ios/IosTestEnvironmentCommand.kt`
   * `IosModelsListCommand`
   * `IosVersionsListCommand`
   * `IosLocalesListCommand`
   * `ProvidedSoftwareListCommand`
   * `NetworkProfilesListCommand`
   * `IosOrientationsListCommand`
   * `IPBlocksListCommand`
   

* `firebase/test/android/AndroidRunCommand.kt`
   * TODO

* `firebase/test/ios/IosRunCommand.kt`
   * TODO

where
* `?` - investigate if implementation is correct or is performing some not necessary operations. 