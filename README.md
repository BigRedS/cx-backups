# coralogix-backups

A quick-and-dirty script to toy with the idea of 'offsite backups' of Coralogix configs.

Run most-simply, will download all your dashboards and alerts to local files. A `cx-backups.d` directory will be created and inside there one more for each thing backed up - `alerts`, `dashboards`, `events2metrics` and `parsing-rules`.

Each of these will contain one JSON file per thing-backed-up; one for each alert, dashboard, e2m rule and parsing-rule group.

Additionally, a `dashboards.yaml` will be written, which attempts to describe the dashboards, and the `dashboard-catalog.json` and `alert-definitions.json` as returned from the API:

```
$ ls
alerts  dashboard-catalog.json  dashboards  dashboards.yaml  events2metrics  parsing-rules
$ ls dashboards | head
Data_Usage::icErszlxXE7ttjqwu-tH_.json
Host_Dashboard::ZerJr6dDM1tJ4JMIgoY1V.json
Kubernetes_Complete_Observability_-_Coralogix_Helmchart_Supported::sTdBFCGXxLo755E7QzeEg.json
Kubernetes_Dashboard_-_Legacy_-_KSM,_cAdvisor,nodemetrics_Supported::cqfOPaj0KXtvb3oUQtMa8.json
Logs_Classification::5QHVm6qhAjZk-mePVuIJQ.json
Logs_Overview::QAI_WrV6Lg09vUr7Na7ha.json
Long_Term_Trends::WrC5CKE8Td0YgajwHK10f.json
Mapping_Exceptions::F8w9eoxtIhtMJI44oBt6F.json
Storage_Pi::jEv2PulBFLW2QCOUlJP41.json
broken_test::4cunUCamVxgCPobCUmK78.json
```

I've not yet needed it, so there isn't currently a cx-restore tool :D

# Git version-tracking

This script can clone a git repo, dump the data into that, commit the changes and push them up. This is is intended to be run as a periodic job so as to be able to track changes - see when that value was set, or what that was set to six weeks ago or something.

Here, I can see that I changed one of the widgets on my Long Term Trends dashboard to use the archive, not frequent-search:

```
$ git diff --unified=0 HEAD~1
diff --git a/dashboard-catalog.json b/dashboard-catalog.json
index 15c4715..6f6db3e 100644
--- a/dashboard-catalog.json
+++ b/dashboard-catalog.json
@@ -111 +111 @@
-      "updateTime": "2025-08-19T09:56:19.109937Z"
+      "updateTime": "2025-08-19T09:58:37.082889Z"
diff --git a/dashboards/Long_Term_Trends::WrC5CKE8Td0YgajwHK10f.json b/dashboards/Long_Term_Trends::WrC5CKE8Td0YgajwHK10f.json
index f82841e..a4baf3f 100644
--- a/dashboards/Long_Term_Trends::WrC5CKE8Td0YgajwHK10f.json
+++ b/dashboards/Long_Term_Trends::WrC5CKE8Td0YgajwHK10f.json
@@ -45 +45 @@
-                          "dataModeType": "DATA_MODE_TYPE_HIGH_UNSPECIFIED",
+                          "dataModeType": "DATA_MODE_TYPE_ARCHIVE",
@@ -147 +147 @@
-  "updatedAt": "2025-08-19T09:56:19.109937Z",
+  "updatedAt": "2025-08-19T09:58:37.082889Z",
```

# Installation and running

On something Debian based you need `sudo apt-get install python3 python3-requests python3-yaml python3-json`, on anything else try

On anything else, get Python installed and do `pip install -r requirements.txt`

There are no supported arguments; everything is set via environment veriables:

Required:
  * CX_API_KEY - your Coralogix API key for API access. Should be a Personal key, not a send-your-data one.
  * CX_REGION  - the three-character name of the region your CX is in (EU0, EU2, etc.)

Optional:
  * CX_BACKUPS_WORKDIR            - the dir to write backups to. Defaults to ./cx-backups.d unless git is being used where it is instead /tmp/cx-backups
  * CX_BACKUPS_GIT_REPO_URL       - the SSH URL of the git repo to commit to. If unset, git is not used at all
  * CX_BACKUPS_GIT_COMMIT_MESSAGE - A commit message; defaults to 'cx-backups ' appended with the date
