Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/BigData/pages/1449234247/Bronze+New+Joiner+Playbook

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Bronze++ New Joiner Playbook

| Process / Tool | PR Link |
|---|---|
| 1. Read_Access: Databricks - DEV / ITG / PROD | For the databricks read access for all the accounts: we need to send request mail to the project owner/admin (Ray Hechavarria) to add to the project through Zone console so that you will grant access to DBR workspace.<br><br>DEV: https://dataos-dev.cloud.databricks.com/<br><br>Prod workspace: https://dataos-prod.cloud.databricks.com<br><br>Dev workspace: https://dataos-dev.cloud.databricks.com<br><br>For databricks write access:<br><br>ITG/ PROD: https://dataos-prod.cloud.databricks.com/ |
| 2.Write_Access: Databricks - DEV / ITG / PROD | 1.https://github.azc.ext.hp.com/hp-data-platform/dataos-databricks-terragrunt/pull/4351<br><br>1. Go to the following link and make a branch starting from feature\add_<your_name>_for_access,<br>2. Clone the repo and go to your branch<br>3. Go to git_repo : DEV(follow the PR to make the changes)<br>4. Go to git_repo: - ITG/PROD(follow the PRto make the changes)<br>5. Go to git_repo: for adding to groups (follow the PR to make the changes)<br>6. Raise the PR(pull request).<br>7. You would also need to raise a Jira ticket for Databricks access - Jira Portal 32 (Data Access/Tools Request). Below the table, you will get a link to a folder for Jira Ticket examples.<br>8. Once your pull request is approved you can get access. |
| Atlan | 1. Click on this link.<br>2. Click on Start an access request<br>3. Click on Application<br>4. Search for 'Atlan Data Catalog'<br>5. Click on read only access<br>6. Give the reason for access<br>7. Submit the request. |
| Airflow - DEV | https://github.azc.ext.hp.com/hp-data-platform/k8s-dataos-core-dev-2/pull/867<br><br>Follow above process and make changes in file -<br><br>https://airflow.dev.hpdataos.com/<br><br>k8s-dataos-core-dev-2/flux2-blue/airflow-dev/infra/airflow_users.yaml<br><br>For Airflow, you would also need to raise a Jira ticket for Databricks access - Jira Portal 32. Below the table, you will get a link to a folder for Jira Ticket examples. |
| Airflow - ITG /PROD | https://github.azc.ext.hp.com/hp-data-platform/k8s-dataos-core-prod/pull/833<br><br>Follow above process and make changes in files -<br><br>https://airflow.stg.hpdataos.com/<br><br>k8s-dataos-core-prod/flux2-blue/airflow-itg/infra/airflow_users.yaml - ITG<br><br>k8s-dataos-core-dev-2/flux2-blue/airflow-prod/infra/airflow_users.yaml - PROD<br><br>https://airflow.hpdataos.com/<br><br>https://github.azc.ext.hp.com/hp-data-platform/k8s-dataos-core-prod/blob/workspaceadd_sulochana_tandra_for_access/flux2-blue/airflow-itg/infra/airflow_users.yaml<br><br>There are 3 places: onecloud-admin, onecloud-viewer, onecloud-paas. Both users are added under onecloud-admin only |
| Splunk | No PR Required<br><br>In case we don't have Splunk access - follow below procedure<br><br>1. Go to the below link and create portal ticket. This is different from the one where we create requests for AWS, Databricks. It is not DataOS access request / Support ticket. https://hp-jira.external.hp.com/servicedesk/customer/portal/616/create/1662<br>2. Provide the required details as mentioned below.<br><br>Email of requested : Optional<br><br>Tool Name : Splunk<br><br>Splunk Best Practices Agreement : Select the check box Splunk Index or User to Clone :<br><br>Bronze++ Automation Splunk 9.0.4.1<br><br>Splunk Search Head<br><br>Environment: DataOS[https://dataos.splunk.prod.cloud.tools.hp-cia.com/]<br><br>Splunk Permission : Read only<br><br>3. You need to mail your manager for approval and paste it as a comment after creating the link<br>4. Once your ticket is approved you can get access. |
| Codeway / Azure Devops Pipeline | No PR Required<br><br>You would need to raise a Jira ticket for Databricks access - Jira Portal 32. Below the table, you will get a link to a folder for Jira TIcket examples. |
| GitHub | Request posted in support-onboarding for getting added to dataos-data-products<br><br>Or send an email to Isaac Chan for getting added to dataos-data-products org. Once a member is onboarded to this org, Sulochana Tandra can add them to onc cloud team team. After that, they can get write access to the repos. |
| Sharepoint Access | Once we click on a sharepoint link, it will take us to a page where we can request access by sending a short message : "I am new joiner in Bronze++ team. Please grant me access to sharepoint where all team related documents exist"<br><br>Send an email to Stuart, Michael <michael.stuart@hp.com> requesting access for site: https://hp.sharepoint.com/sites/DataOSOperations/ on sharepoint. |
| Wiki | Any admin should be able to add the user with a valid hp email id. Contact: Sulochana Tandra |
| cscrpds aws bucket | Need to raise portal-32 ticket<br><br>To get the access for cscrpds aws bucket which we need to work on OpenSearch dashboard. we need to raised portal-32 ticket, below are the reference portal-32 request tickets.<br><br>https://hp-jira.external.hp.com/servicedesk/customer/portal/32/DATAOS-19428<br><br>https://hp-jira.external.hp.com/servicedesk/customer/portal/32/DATAOS-19427 |
| Zone Console | NA<br><br>URL: https://console.hponecloud.com/projects?ownership=0&phase=-1&usage=0&name=&sort=new_old<br><br>Subscribe to the Tag: DataOS NG IND PROD |

Jira Ticket Examples - New Onboarding Jira Tickets

AWS Access:

To get access to AWS buckets, there are 2 steps:

1. Raise a portal 32 ticket to get access to the account. Eg: https://hp-jira.external.hp.com/servicedesk/customer/portal/32/DATAOS-12369
  - Once above ticket gets approved, one will be able to see the account in AWS Console.

2. Raise a PR to add your email to the account you need access to. Eg: https://github.azc.ext.hp.com/hp-data-platform/dataos-dev-terragrunt/pull/1075
  - Once the above PR is approved, one would be able to switch the role from READONLY to the role for which above PR is raised.

Refer to access.xlsx (below document) to raise access requests.

Access (1).xlsx