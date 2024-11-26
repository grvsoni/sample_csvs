Customer Slug: form3
 
ARR: $782,647.15
 
Customer Temp (Cool, warm, hot, volcanic, etc): Cool
 
Priority: P2
 
Project ["support-escalations project" from this spreadsheet]: Mimir
 
Label ["support-escalations label" from this spreadsheet]: enterprise-metrics
 
Summary of Issue: 
 
They are running Alloy 1.4.3-1
 
The customer, Łukasz, reported that metrics from different Alloy instances are not being deduplicated as expected, resulting in multiple series with the same labels. They confirmed that this issue did not occur previously.
 
https://form3test.grafana.net/explore?schemaVersion=1&panes=%7B%228s1%22:%7B%22datasource%22:%22grafanacloud-prom%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22form3_public_payment_admission_tasks_status%7Bstatus%3D%5C%22completed%5C%22,%20organisation_id%3D%5C%22ddf6ae0a-578a-40f0-a756-5773a869d513%5C%22,%20hash_id%3D%5C%22497483683200797043%5C%22,%20scheme_payment_type%3D%5C%22standingorder%5C%22,%20status%3D%5C%22completed%5C%22,%20task_name%3D%5C%22ledger_posting%5C%22,%20outcome%3D%5C%22passed%5C%22%7D%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22grafanacloud-prom%22%7D,%22editorMode%22:%22code%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%221731741019713%22,%22to%22:%221731811387848%22%7D%7D%7D&orgId=1
 
When running the PromQL,
form3_public_payment_admission_tasks_status{status="completed", organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513", hash_id="497483683200797043", scheme_payment_type="standingorder", status="completed", task_name="ledger_posting", outcome="passed"}
 
They are getting results in series as:
form3_public_payment_admission_tasks_status{__replica__="alloy-metrics-84d6c5f956-dk76m",cluster="625458760271544958",hash_id="497483683200797043",metrics_api="public",organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513",outcome="passed",scheme_payment_type="standingorder",status="completed",task_name="ledger_posting"}

form3_public_payment_admission_tasks_status{__replica__="alloy-metrics-84d6c5f956-r5zbx",cluster="625458760271544958",hash_id="497483683200797043",metrics_api="public",organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513",outcome="passed",scheme_payment_type="standingorder",status="completed",task_name="ledger_posting"}

form3_public_payment_admission_tasks_status{cluster="625458760271544958",hash_id="497483683200797043",metrics_api="public",organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513",outcome="passed",scheme_payment_type="standingorder",status="completed",task_name="ledger_posting"}
 
ENVIRONMENT DETAILS
On-Prem or HG: HG
(On-prem) Platform (Linux, Windows, K8s, AMG, etc): 
(Cloud) Instance Slug: form3test
(HG) Permission to access instance (Y/N): Y
(HG) Link to affected page: https://form3test.grafana.net/explore?schemaVersion=1&panes=%7B%228s1%22:%7B%22datasource%22:%22grafanacloud-prom%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22form3_public_payment_admission_tasks_status%7Bstatus%3D%5C%22completed%5C%22,%20organisation_id%3D%5C%22ddf6ae0a-578a-40f0-a756-5773a869d513%5C%22,%20hash_id%3D%5C%22497483683200797043%5C%22,%20scheme_payment_type%3D%5C%22standingorder%5C%22,%20status%3D%5C%22completed%5C%22,%20task_name%3D%5C%22ledger_posting%5C%22,%20outcome%3D%5C%22passed%5C%22%7D%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22grafanacloud-prom%22%7D,%22editorMode%22:%22code%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%221731741019713%22,%22to%22:%221731811387848%22%7D%7D%7D&orgId=1
 
Grafana Version: 11.4.0-79146
 
License: Contracted Cloud Advanced

Product: Mimir
 
Component: Querier
(if applicable) Tenant ID: 91931 - form3test-prom
(if applicable) Version of component (plugin, data source, etc):
 
Support Investigation:
For now, I have given them a workaround
max by (cluster, hash_id, metrics_api, organisation_id, outcome, scheme_payment_type, status, task_name) (
  form3_public_payment_admission_tasks_status{status="completed", organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513", hash_id="497483683200797043", scheme_payment_type="standingorder", status="completed", task_name="ledger_posting", outcome="passed"}
)
This query will select only one series per unique combination of the specified labels, effectively deduplicating the results.
 
If you can see the cluster label in the metrics, it's likely that alloy is adding it correctly. Mimir does not add the cluster label; it expects this label to be present in the incoming metrics. If both __replica__ and cluster labels are present and configured correctly, Mimir should be able to deduplicate the metrics properly.
 
In the series below we see, 
 
form3_public_payment_admission_tasks_status{__replica__="alloy-metrics-84d6c5f956-dk76m",cluster="625458760271544958",hash_id="497483683200797043",metrics_api="public",organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513",outcome="passed",scheme_payment_type="standingorder",status="completed",task_name="ledger_posting"}
form3_public_payment_admission_tasks_status{__replica__="alloy-metrics-84d6c5f956-r5zbx",cluster="625458760271544958",hash_id="497483683200797043",metrics_api="public",organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513",outcome="passed",scheme_payment_type="standingorder",status="completed",task_name="ledger_posting"}
form3_public_payment_admission_tasks_status{cluster="625458760271544958",hash_id="497483683200797043",metrics_api="public",organisation_id="ddf6ae0a-578a-40f0-a756-5773a869d513",outcome="passed",scheme_payment_type="standingorder",status="completed",task_name="ledger_posting"}
 
The first two metrics have both the __replica__ and cluster labels, The third metric is missing the __replica__ label but retains the cluster label. This pattern is consistent with Grafana Mimir's deduplication process.
Mimir receives metrics from multiple replicas (the first two metrics).
It uses the __replica__ and cluster labels to identify and deduplicate these metrics.
After deduplication, Mimir drops the __replica__ label but keeps the cluster label.
The third metric you see is likely the result of this process. But I would expect the Mimir to drop the first two duplicate series as well and not result in query result.
 
It seems to be a problem with deduplication in the query.
 
We have also recommended them to use Alloy clustering as then they don't need to use dedupe at all since it auto-distributes targets and only sends one copy instead of multiple. We also told them it would also reduce their resource usage since they wouldn't be capturing everything 2+ times.
The customer is considering alloy clustering for resource-wise reasons (as in one of the clusters Alloy needs up to 17-18 GB of RAM). However, we're afraid that this will come with extra maintenance and problems on our side, so we'd like to fix the deduplication first.
 
Can we help them fix the deduplication issue with their current setup as they confirmed that this issue did not occur previously?
 
Attaching Alloy config with all prometheus.* components as alloy-metrics.alloy
 
 
Resources used (docs, ops.grafana, slack threads, github issues, source code, google search): 
https://raintank-corp.slack.com/archives/C9KL8THCY/p1732545142882999
 
 
 
Steps to reproduce (if possible):
Go to https://form3test.grafana.net/explore?schemaVersion=1&panes=%7B%228s1%22:%7B%22datasource%22:%22grafanacloud-prom%22,%22queries%22:%5B%7B%22refId%22:%22A%22,%22expr%22:%22form3_public_payment_admission_tasks_status%7Bstatus%3D%5C%22completed%5C%22,%20organisation_id%3D%5C%22ddf6ae0a-578a-40f0-a756-5773a869d513%5C%22,%20hash_id%3D%5C%22497483683200797043%5C%22,%20scheme_payment_type%3D%5C%22standingorder%5C%22,%20status%3D%5C%22completed%5C%22,%20task_name%3D%5C%22ledger_posting%5C%22,%20outcome%3D%5C%22passed%5C%22%7D%22,%22range%22:true,%22instant%22:true,%22datasource%22:%7B%22type%22:%22prometheus%22,%22uid%22:%22grafanacloud-prom%22%7D,%22editorMode%22:%22code%22,%22legendFormat%22:%22__auto%22%7D%5D,%22range%22:%7B%22from%22:%221731741019713%22,%22to%22:%221731811387848%22%7D%7D%7D&orgId=1
See the duplicate series.
 
 
Diagnostic data (configs, screenshots, videos, gong, dashboard JSON, Panel DataFrame, HAR file, debug logs):
alloy-metrics.alloy [Alloy config with all prometheus.* components]
 
 
