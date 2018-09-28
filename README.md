# Alerting Framework for AdWords

## Deprecation Notice

Due to low activity in the Alerting Framework for AdWords project, we've decided
to deprecate it on September 28, 2018. There will not be any updates for AdWords
API version migrations or issue tracking after this date. Please clone or fork
this repository and migrate to the latest AdWords API version if you intend to
continue using the code. This repository will be removed on or after 12th January 2019.

We encourage you to own and maintain this project. Please fork or clone the
repository and follow the [Google Ads Developer Blog](https://ads-developers.googleblog.com/)
for notifications of upcoming API migrations.

Your feedback is valuable to us in shaping our products. If you have comments or
suggestions, please raise these on the project’s issue page.

All the best for your future ads integrations.

Google Ads team

## Overview

Alerting Framework for AdWords is an open source Java framework for large
scale AdWords API alerting. It's based on the [Java AdWords API client
library] (https://github.com/googleads/googleads-java-lib). This framework
is capable of downloading AdWords report data and combining with other
data feeds, and processing them according to specified alert rules and
alert actions in the configuration. You can use our sample alerts to
explore how it works or set up your own fully customized logic. The alerts
available through this tool cater to both new and experienced users. Users
can set up simple alerts with sample alert entities, or implement custom
alert entities through the interfaces and plug into the system.

## Quick Start

### Prerequisites

You will need Java, Maven and MySQL installed before configuring the project.

### Build the project using Maven

You can clone the git repository into a local folder by executing the following
command:
```
$ git clone https://github.com/googleads/adwords-alerting
```

And compile the Alerting Framework for AdWords using Maven by executing the
following command:
```
$ mvn compile dependency:copy-dependencies package
```

### Configure Alerting Framework for AdWords
```
vi java/resources/aw-alerting-sample.properties
```
 - Fill in the developer token, client ID / secret, manager account ID and
   refresh token.

```
vi java/resources/aw-alerting-alerts-sample.json
```
 - This is the JSON configuration of the alerts that you want to run. It's
   referred from the .properties file in the same folder.

## Run the project

```
java -Xmx4G -jar aw-alerting.jar -file <file>

Arguments:

 -accountIdsFile <file>  ONLY run the alerting logic for client customer IDs
                         specified in the file

 -debug                  Display all the debug information. If option 'verbose'
                         is present, all the information will be displayed on
                         the console as well

 -file <file>            The properties file (please refer to the file
                         ./aw-alerting-sample.properties as an example)

 -help                   Display full help information
```

## Implement custom alert report downloader

Alert report downloader is responsible for downloading report data for further
processing (e.g. apply rules and actions). We provide an implementation
[AwqlReportDownloader](https://github.com/googleads/aw-alerting/blob/master/java/com/google/api/ads/adwords/awalerting/sampleimpl/downloader/AwqlReportDownloader.java)
that downloads report data using [AWQL](https://developers.google.com/adwords/api/docs/guides/awql).
Custom alert report downloader:
 - Should derive from
   ``com.google.api.ads.adwords.awalerting.downloader.AlertReportDownloader``,
   and
 - Must have a constructor with a JsonObject parameter to work properly.

## Implement custom alert rules

Alert rules are responsible for:

 - Defining a list of field names to extend in the report
 - Determining a list of field values to extend for each report entry
 - Determining whether each report entry should be skipped from result alerts

Custom alert rules:
 - Should derive from
   ``com.google.api.ads.adwords.awalerting.rule.AlertRule``, and
 - Must have a constructor with a JsonObject parameter to work properly, and
 - Must be stateless, since the same instance will be shared among multiple
   threads.

## Implement custom alert actions

Alert actions are responsible for processing each report entry, and:

 - Performing some action immediately, or
 - Recording some info and performing aggregated action at the end

Custom alert actions:
 - Should derive from
   ``com.google.api.ads.adwords.awalerting.action.AlertAction``, and
 - Must have a constructor with a JsonObject parameter to work properly, and
 - Must NOT modify report entries (since an report entry may be processed by
   multiple alert actions). Any modification should be done by AlertRules.

## Plug custom report downloader, alert rules and alert actions

Just edit the JSON configuration file:

 - Under ``ReportDownloader``, put class name of the custom alert
   report downloader in ``ClassName`` field, along with any other
   parameters that will be passed to the custom alert report downloader's
   constructor. For example:
 ```
    "ReportDownloader": {
      "ClassName": "AwqlReportDownloader",
      "ReportQuery": {
        "ReportType": "KEYWORDS_PERFORMANCE_REPORT",
        "Fields": "ExternalCustomerId,Id,Criteria,Impressions,Ctr",
        "Conditions": "Impressions > 100 AND Ctr < 0.05",
        "DateRange": "YESTERDAY"
      }
    }
 ```

 - Under ``Rules``, put class name of the custom alert rule in
   ``ClassName`` field, along with other parameters that will be
   passed to the custom alert rule's constructor. For example:
 ```
    "Rules": [
      {
        "ClassName": "AddAccountManager"
      },
      {
        "ClassName": "AddAccountMonthlyBudget"
      }
    ]
 ```

 - Under ``Actions``, put class name of the custom alert rule in
   ``ClassName`` field, along with other parameters that will be
   passed to the custom alert rule's constructor. For example:
 ```
    "Actions": [
      {
        "ClassName": "SimpleConsoleWriter"
      },
      {
        "ClassName": "PerAccountManagerEmailSender",
        "Subject": "Low impression accounts",
        "CC": "abc@example.com,xyz@example.com"
      }
    ]
 ```

### Fine print
Pull requests are very much appreciated. Please sign the
[Google Code contributor license agreement]
(https://cla.developers.google.com/about/google-individual)
(There is a convenient online form) before submitting.

<dl>
  <dt>Copyright</dt>
  <dd>Copyright © 2015 Google, Inc.</dd>
  <dt>License</dt>
  <dd>Apache 2.0</dd>
  <dt>Limitations</dt>
  <dd>This is example software, use with caution under your own risk.</dd>
</dl>
