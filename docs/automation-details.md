# FOI Management – Automation Details

This document lists the Power Automate flows used in the FOI Management Suite.  

| **Flow Name**                     | **Summary**                                                                 | **Trigger**                                                        | **Connectors Used**           |
|-----------------------------------|-----------------------------------------------------------------------------|--------------------------------------------------------------------|-------------------------------|
| **Assign FOI Request to FOI Officers team** | Assigns new FOI request to the FOI Officers Team.         | Dataverse – When a row is added (Create)                           | Dataverse                     |
| **FOI Responded Notification** | Notify the requester (contact) of the FOI Request's outcome.         | Dataverse — When a row is modified                           | Dataverse , Gmail                    |
| **Auto‐Create Case from FOI Escalation** | Create a new Case record and links back to FOI Request if Request's "Escalated to Case" option is "Yes".         | Dataverse — When a row is modified                           | Dataverse                |
| **Set Case Status** | Update Case' "Status" and "Status Reason" fields when "Close Category" is updated (primarily via BPF).         | Dataverse — When a row is modified                           | Dataverse                |
| **SLA Breach Notification**       | Sends an email alert to the Case's Assigned Officer when SLA is about to breach and breached.            | Dataverse – When a row is modified                                 | Dataverse            |
| **Notify on Response File Upload**  | Uploads a formal case document to SharePoint and also creates a Request folder if not yet exists. Then notifies both the Assigned Officer and Requester | Dataverse – When a row is modified                                           | Dataverse, Microsoft 365 Outlook, Office 365 Users, SharePoint       |
| **Create FOI Request Folder** | Creates a dedicated SharePoint folder for each new FOI request and links the path to the FOI Request record.              | Power Apps (Canvas app) calls the flow                           | Dataverse, Power Apps, SharePoint         |
| **Create FOI Request Document** | Create the uploaded document in Canvas App into SharePoint library.              | Power Apps (Canvas app) calls the flow                           | Dataverse, Power Apps, SharePoint         |
| **Delete FOI Request Document** | Delete the selected document in Canvas App from SharePoint library.              | Power Apps (Canvas app) calls the flow                           | Dataverse, Power Apps, SharePoint         |