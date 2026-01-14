# FOI Management – Automation Details

This document lists the automation components used in the FOI Management Suite.

## Power Automate Flows (Application-Level Automation)

This section lists Power Automate flows used for FOI case lifecycle automation, notifications, and document handling.

| **Flow Name**                     | **Summary**                                                                 | **Trigger**                                                        | **Connectors Used**           |
|-----------------------------------|-----------------------------------------------------------------------------|--------------------------------------------------------------------|-------------------------------|
| **Run FOI Request Migration** |  Batch processes staged FOI Request Import data with validation, idempotent create/update logic, and execution outcome tracking.         | Manually triggered                           | Dataverse       |
| **Audit Request Owner Changes** |   Logs FOI Request owner changes.             | Dataverse - When a row is modified                           | Dataverse       |
| **Audit Request Status Changes** |    Logs FOI Request' request status changes.           | Dataverse - When a row is modified                           | Dataverse       |
| **Audit Request Due Date Changes** |  Logs FOI Request due date changes.             | Dataverse - When a row is modified                           | Dataverse       |
| **Assign FOI Request to FOI Officers team** | Assigns new FOI request to the FOI Officers Team.         | Dataverse - When a row is added (Create)                           | Dataverse                     |
| **FOI Responded Notification** | Notifies the requester (contact) of the FOI Request's outcome.         | Dataverse - When a row is modified                           | Dataverse, Gmail                    |
| **Auto‐Create Case from FOI Escalation** | Creates a new Case record and links back to FOI Request if Request's "Escalated to Case" option is "Yes".         | Dataverse - When a row is modified                           | Dataverse                |
| **Set Case Status** | Update Case' "Status" and "Status Reason" fields when "Close Category" is updated (primarily via BPF).         | Dataverse - When a row is modified                           | Dataverse                |
| **SLA Breach Notification**       | Sends an email alert to the Case's Assigned Officer when SLA is about to breach and breached.            | Dataverse - When a row is modified                                 | Dataverse            |
| **Notify on Response File Upload**  | Uploads a formal case document to SharePoint and also creates a Request folder if not yet exists. Then notifies both the Assigned Officer and Requester. | Dataverse - When a row is modified                                           | Dataverse, Microsoft 365 Outlook, Office 365 Users, SharePoint       |
| **Create FOI Request Folder** | Creates a dedicated SharePoint folder for each new FOI request and links the path to the FOI Request record.              | Power Apps (Canvas app) calls the flow                           | Dataverse, Power Apps, SharePoint         |
| **Create FOI Request Document** | Creates the uploaded document in Canvas App into SharePoint library.              | Power Apps (Canvas app) calls the flow                           | Dataverse, Power Apps, SharePoint         |
| **Delete FOI Request Document** | Deletes the selected document in Canvas App from SharePoint library.              | Power Apps (Canvas app) calls the flow                           | Dataverse, Power Apps, SharePoint         |


## Logic Apps (Integration & Asynchronous Processing)

This section lists Azure Logic Apps used for inbound integrations and asynchronous, non-blocking processing that sits outside the core FOI application lifecycle.

| **Logic App Name** | **Summary** | **Trigger** | **Downstream Systems** |
|-------------------|-------------|-------------|------------------------|
| **Email Ingestion Logic App** | Processes inbound FOI emails, stages metadata as FOI Request Import records, uploads attachments to SharePoint, and writes Integration Logs | Outlook / Shared Mailbox | Dataverse, SharePoint |
| **FOI Events Consumer Logic App** | Consumes validated FOI events from Service Bus and performs downstream processing | Service Bus (Queue/Topic) | Dataverse |
