Source tag : Bronze++NewJoinerbook 1.pdf

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Bronze++ New Joiner Playbook

| Process / Tool | PR Link | |
|---|---|---|
| 1. Read_Access: Databricks - DEV / ITG / PROD | | For the databricks read access for all the accounts: we need to send request mail to the project owner/admin (Ray Hechavarria) to add to the project through Zone console so that you will grant access to DBR workspace.<br><br>DEV:<br>Prod workspace:<br>Dev workspace:<br><br>For databricks write access:<br>ITG/ PROD: |
| 2.Write_Access: Databricks - DEV / ITG / PROD | 1. | 1. Go to the following link and make a branch starting from feature\add_<your_name>_for_access,<br>2. Clone the repo and go to your branch<br>3. Go to git_repo : DEV(follow the PR to make the changes)<br>4. Go to git_repo: - ITG/PROD(follow the PRto make the changes)<br>5. Go to git_repo: for adding to groups (follow the PR to make the changes)<br>6. Raise the PR(pull request).<br>7. You would also need to raise a Jira ticket for Databricks access - Jira Portal 32 (Data Access/Tools Request). Below the table, you will get a link to a folder for Jira Ticket examples.<br>8. Once your pull request is approved you can get access. |
| Atlan | | 1. Click on this link.<br>2. Click on Start an access request<br>3. Click on Application<br>4. Search for 'Atlan Data Catalog'<br>5. Click on read only access<br>6. Give the reason for access<br>7. Submit the request. |
| Airflow - DEV | | Follow above process and make changes in file -<br>k8s-dataos-core-dev-2/flux2-blue/airflow-dev/infra/airflow_users.yaml<br><br>For Airflow, you would also need to raise a Jira ticket for Databricks access - Jira Portal 32. Below the table, you will get a link to a folder for Jira Ticket examples. |
| Airflow - ITG /PROD | | Follow above process and make changes in files -<br>k8s-dataos-core-prod/flux2-blue/airflow-itg/infra/airflow_users.yaml - ITG<br>k8s-dataos-core-dev-2/flux2-blue/airflow-prod/infra/airflow_users.yaml - PROD<br><br>There are 3 places: onecloud-admin, onecloud-viewer, onecloud-paas. Both users are added under onecloud-admin only |
| Splunk | No PR Required | In case we don't have Splunk access - follow below procedure<br><br>1. Go to the below link and create portal ticket. This is different from the one where we create requests for AWS, Databricks. It is not DataOS access request / Support ticket.<br>2. Provide the required details as mentioned below.<br>   Email of requested : Optional<br>   Tool Name : Splunk<br>   Splunk Best Practices Agreement : Select the check box Splunk Index or User to Clone :<br>   Bronze++ Automation | Splunk 9.0.4.1 (hp-cia.com)<br>   Splunk Search Head<br>   Environment: DataOS<br>   Splunk Permission : Read only<br>3. You need to mail your manager for approval and paste it as a comment after creating the link<br>4. Once your ticket is approved you can get access. |
| Codeway / Azure Devops Pipeline | No PR Required | You would need to raise a Jira ticket for Databricks access - Jira Portal 32. Below the table, you will get a link to a folder for Jira TIcket examples. |
| GitHub | | Request posted in support-onboarding for getting added to dataos-data-products<br><br>Or send an email to Isaac Chan for getting added to dataos-data-products org. Once a member is onboarded to this org, Sulochana Tandra can add them to onc cloud team team. After that, they can get write access to the repos. |
| Sharepoint Access | | Once we click on a sharepoint link, it will take us to a page where we can request access by sending a short message : "I am new joiner in Bronze++ team. Please grant me access to sharepoint where all team related documents exist"<br><br>Send an email to Stuart, Michael requesting access for site: on sharepoint. |
| Wiki | | Any admin should be able to add the user with a valid hp email id. Contact: Sulochana Tandra |
| cscrpds aws bucket | Need to raise portal-32 ticket | To get the access for cscrpds aws bucket which we need to work on OpenSearch dashboard. we need to raised portal-32 ticket, below are the reference portal-32 request tickets. |
| Zone Console | NA | Subscribe to the Tag: DataOS NG IND PROD |

Jira Ticket Examples - New Onboarding Jira Tickets

AWS Access:

To get access to AWS buckets, there are 2 steps:

1. Raise a portal 32 ticket to get access to the account.
   - Once above ticket gets approved, one will be able to see the account in AWS Console.
2. Raise a PR to add your email to the account you need access to.
   - Once the above PR is approved, one would be able to switch the role from READONLY to the role for which above PR is raised.

Refer to access.xlsx (below document) to raise access requests.

Access (1).xlsx