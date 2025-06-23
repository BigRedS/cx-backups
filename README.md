# coralogix-backups

A quick-and-dirty script to toy with the idea of 'offsite backups' of Coralogix configs.

Run most-simply, will download all your dashboards and alerts to local files. A `cx-backups.d` directory will be created and inside there two more: `alerts` and `dashboards`.

In the dashboards directory, a JSON file will be written for each dashboard containing its definition, named `<dashboard-id>::<dashboard_name>`, plus one called `catalog.json` that records which boards go in which folders.

In the alerts directory, a JSON file will be written for each alert containing its definition, named `<alert-id>::<alert-name>` plus one called `definitions.json` that contains all of them in one file. as returned by the API.

    find cx-backups.d/
    cx-backups.d/
    cx-backups.d/dashboards
    cx-backups.d/dashboards/F8w9eoxtIhtMJI44oBt6F::Mapping_Exceptions.json
    x-backups.d/dashboards/icErszlxXE7ttjqwu-tH_::Data_Usage.json
    cx-backups.d/dashboards/QAI_WrV6Lg09vUr7Na7ha::Logs_Overview.json
    cx-backups.d/dashboards/5QHVm6qhAjZk-mePVuIJQ::Logs_Classification.json
    cx-backups.d/dashboards/catalog.json
    cx-backups.d/dashboards/ZerJr6dDM1tJ4JMIgoY1V::Host_Dashboard.json
    cx-backups.d/dashboards/sTdBFCGXxLo755E7QzeEg::Kubernetes_Complete_Observability_-_Coralogix_Helmchart_Supported.json
    cx-backups.d/alerts
    cx-backups.d/alerts/dea85261-2572-472a-9ad0-a0c82cd9db29::K8s_-_Pod_Scheduling_Failure.json
    cx-backups.d/alerts/199f203c-94be-44d3-96db-febc8718101e::K8s_-_Pod_Failure_Detected.json
    cx-backups.d/alerts/adfbab76-5f47-4f1d-b862-fe0f8dcf7a61::K8s_-_Percentage_of_Healthy_Pods_Below_44%.json
    cx-backups.d/alerts/d8da10d6-22fc-4042-9a51-d44bd5d390c3::K8s_-_Node_Memory_Utilization_Higher_Than_80%.json
    cx-backups.d/alerts/10d1f9e9-b499-47a2-bfa2-b25f12bf6327::K8s_-_Node_CPU_Utilization_Above_80%.json
    cx-backups.d/alerts/3a3a3616-cca6-49f4-8acb-2dc842dcd57a::K8s_-_DaemonSet_MisScheduled_Pods_-_All_namespace.json
    cx-backups.d/alerts/b59193a0-90f0-49fa-99ae-d2a2b5799f37::K8s_-_Volume_Mount_Failure.json
    cx-backups.d/alerts/28d603e7-f2d5-44ef-845d-a43c23e0d33a::K8s_-_Node_Approaching_Out_of_Memory_(Oom).json
    cx-backups.d/alerts/definitions.json

Automated restoration is not attempted here; the idea is you could refer back to these, perhaps import a dashboard json export if you want, and/or use any of these when building out Terraform configs. The JSON docs are as exported by the UI, and as exported and as expected by the APIs.

I expect to export more types of coralogix thing (e2m configs, parsing rules etc.) as and when it becomes useful to me. Half the point of this script is to show you how easy it is to write your own, but feel free to ask if you'd like to wait for me to do it :D

# Version-tracking

With the right config, instead of creating and working in a `./cx-backups.d` directory, the script can clone a git repo, dump the data into that, commit the changes and push them up. This is is intended to be run as a periodic job so as to be able to track changes - see when that value was set, or what that was set to six weeks ago or something.

# Installation and running

All you should need to do to get this working is:

    apt-get install python3 grpcurl

There are no supported arguments; everything is set via environment veriables:

Required:
  * CX_API_KEY - your Coralogix API key for API access. Should be a Personal key, not a send-your-data one.
  * CX_REGION  - the three-character name of the region your CX is in (EU0, EU2, etc.)

Optional:
  * CX_BACKUPS_WORKDIR            - the dir to write backups to. Defaults to ./cx-backups.d unless git is being used where it is instead /tmp/cx-backups
  * CX_BACKUPS_GIT_REPO_URL       - the SSH URL of the git repo to commit to. If unset, git is not used at all
  * CX_BACKUPS_GIT_COMMIT_MESSAGE - A commit message; defaults to 'cx-backups ' appended with the date
