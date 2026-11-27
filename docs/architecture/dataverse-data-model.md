# Dataverse Data Model

## 1. Overview

This document describes the initial Dataverse data model for the Enterprise Service Operations Hub.

Dataverse will act as the central data source for service requests, assignments, approvals and SLA tracking. The data will be used by the employee-facing Canvas app, the operations model-driven app, Power Automate flows and Power BI reports.

The model is built around the Service Request table, with separate tables for information that needs to be tracked over time.

**Publisher prefix:** `eso`

## 2. Tables and Columns

### 2.1 Service Request

**Logical name:** `eso_servicerequest`  
**Ownership:** User or team owned

| Column               | Type                   | Notes                                                              |
|----------------------|------------------------|--------------------------------------------------------------------|
| Request Number       | Autonumber             | Format: `SR-{SEQNUM:6}`. Primary name column and unique reference. |
| Title                | Text                   | Short summary of the request.                                      |
| Description          | Multiline text         | Details provided by the employee.                                 |
| Requester            | Lookup to User         | Dataverse system user who raised the request.                      |
| Department           | Lookup to Department   | Requester's department; optional initially.                       |
| Category             | Lookup to Request Category | Category selected during submission.                           |
| Priority             | Choice                 | Critical, High, Medium, Low.                                       |
| Status               | Choice                 | Draft, Submitted, Assigned, In Progress, Pending Approval, Resolved, Closed, Rejected. |
| Submitted On         | Date and time          | Populated when the request is submitted.                           |
| First Response On    | Date and time          | Records the first response from the operations team.               |
| Resolved On          | Date and time          | Populated when the request is resolved.                            |
| Closed On            | Date and time          | Populated when the request is closed.                              |
| Current Support Team | Lookup to Team         | Team currently responsible for the request.                       |
| Current Agent        | Lookup to User         | Agent currently handling the request.                              |

The Owner field is separate from Current Support Team and Current Agent. Ownership controls record access, while the assignment fields identify who is responsible for the work.

### 2.2 Request Category

**Logical name:** `eso_requestcategory`  
**Ownership:** Organisation owned

| Column               | Type                         | Notes                                                |
|----------------------|------------------------------|------------------------------------------------------|
| Category Name        | Text                         | Display name of the category.                        |
| Category Code        | Text                         | Unique category identifier.                          |
| Description          | Multiline text               | Description of the category.                         |
| Parent Category      | Lookup to Request Category   | Optional self-reference for category hierarchy.     |
| Approval Required    | Yes/No                       | Indicates whether requests in this category need approval. |
| Default Support Team | Lookup to Team               | Default team for request routing.                    |
| Is Active            | Yes/No                       | Controls availability for new requests.              |

### 2.3 Department

**Logical name:** `eso_department`  
**Ownership:** Organisation owned

| Column           | Type               | Notes                                      |
|------------------|--------------------|--------------------------------------------|
| Department Name  | Text               | Department name.                           |
| Department Code  | Text               | Unique department identifier.              |
| Department Manager | Lookup to User   | Manager responsible for the department.    |
| Description      | Multiline text     | Optional department details.               |
| Is Active        | Yes/No             | Indicates whether the department is active. |

### 2.4 Request Assignment

**Logical name:** `eso_requestassignment`  
**Ownership:** User or team owned

| Column             | Type                     | Notes                                                |
|--------------------|--------------------------|------------------------------------------------------|
| Assignment Number  | Autonumber               | Unique assignment reference.                         |
| Service Request    | Lookup to Service Request | Related service request.                            |
| Assigned Team      | Lookup to Team           | Team receiving the request.                          |
| Assigned Agent     | Lookup to User           | Agent assigned to the request.                       |
| Assigned By        | Lookup to User           | User who initiated the assignment.                   |
| Assigned On        | Date and time            | Assignment date and time.                            |
| Assignment End Date | Date and time           | Populated when the assignment ends.                  |
| Assignment Status  | Choice                   | Active, Reassigned, Completed, Cancelled.            |
| Assignment Notes   | Multiline text           | Optional assignment or handover notes.               |

A new record will be created for each reassignment. This preserves assignment history instead of overwriting the previous agent or team.

### 2.5 Request Status History

**Logical name:** `eso_requeststatushistory`  
**Ownership:** User or team owned

| Column          | Type                     | Notes                                      |
|-----------------|--------------------------|--------------------------------------------|
| History Number  | Autonumber               | Unique history reference.                 |
| Service Request | Lookup to Service Request | Related request.                           |
| Previous Status | Choice                   | Status before the change.                  |
| New Status      | Choice                   | Status after the change.                  |
| Changed By      | Lookup to User           | User responsible for the change.          |
| Changed On      | Date and time            | Date and time of the change.               |
| Change Reason   | Multiline text           | Optional reason for the change.            |

A history record should be created for every status change through controlled automation or server-side logic.

### 2.6 SLA Configuration

**Logical name:** `eso_slaconfiguration`  
**Ownership:** Organisation owned

| Column                   | Type                       | Notes                                         |
|--------------------------|----------------------------|-----------------------------------------------|
| SLA Name                 | Text                       | Name of the SLA configuration.               |
| SLA Code                 | Text                       | Unique configuration identifier.             |
| Priority                 | Choice                     | Critical, High, Medium, Low.                  |
| Category                 | Lookup to Request Category | Optional category-specific SLA rule.         |
| Response Target Minutes  | Whole number               | Allowed time for first response.             |
| Resolution Target Minutes | Whole number              | Allowed time for resolution.                 |
| Is Active                | Yes/No                     | Indicates whether the configuration is active. |
| Effective From           | Date only                  | Date when the configuration becomes applicable. |
| Effective To             | Date only                  | Optional expiry date.                        |

Initial sample targets for development:

| Priority | Response Target | Resolution Target |
|----------|-----------------|-------------------|
| Critical | 30 minutes      | 240 minutes       |
| High     | 60 minutes      | 480 minutes       |
| Medium   | 240 minutes     | 1,440 minutes     |
| Low      | 480 minutes     | 2,880 minutes     |

These are sample project values. The initial implementation will use elapsed minutes. Business hours and holiday calendars can be introduced as a later enhancement.

### 2.7 SLA Tracking

**Logical name:** `eso_slatracking`  
**Ownership:** User or team owned

| Column              | Type                       | Notes                                           |
|---------------------|----------------------------|-------------------------------------------------|
| SLA Tracking Number | Autonumber                 | Unique tracking reference.                      |
| Service Request     | Lookup to Service Request  | Request being monitored.                        |
| SLA Configuration  | Lookup to SLA Configuration | SLA configuration applied.                    |
| SLA Cycle           | Whole number               | Identifies the SLA cycle, including reopened requests. |
| SLA Start Time      | Date and time              | Start of SLA measurement.                       |
| Response Due On     | Date and time              | Calculated response deadline.                   |
| Resolution Due On   | Date and time              | Calculated resolution deadline.                 |
| First Response On   | Date and time              | Actual first response date and time.            |
| Resolved On         | Date and time              | Actual resolution date and time.                |
| Response Breached   | Yes/No                     | Indicates whether the response target was breached. |
| Resolution Breached | Yes/No                     | Indicates whether the resolution target was breached. |
| SLA Status          | Choice                     | Active, Paused, Met, Breached, Cancelled.        |

SLA Tracking stores the result for each request, while SLA Configuration stores the target rules. Separate cycles allow reopened requests to retain their previous SLA history.

### 2.8 Approval

**Logical name:** `eso_approval`  
**Ownership:** User or team owned

| Column            | Type                     | Notes                                      |
|-------------------|--------------------------|--------------------------------------------|
| Approval Number   | Autonumber               | Unique approval reference.                 |
| Service Request   | Lookup to Service Request | Request requiring approval.               |
| Approver User     | Lookup to User           | User responsible for the decision.         |
| Approval Status   | Choice                   | Pending, Approved, Rejected, Cancelled.    |
| Requested On      | Date and time            | Date and time approval was requested.      |
| Responded On      | Date and time            | Date and time of the decision.             |
| Approval Comments | Multiline text           | Comments from the approver.                |

One record will be created per approver per approval stage. Power Automate Approvals will handle the approval interaction.

### 2.9 Request Comment

**Logical name:** `eso_requestcomment`  
**Ownership:** User or team owned

| Column                | Type                     | Notes                                                  |
|-----------------------|--------------------------|--------------------------------------------------------|
| Comment Number        | Autonumber               | Unique comment reference.                              |
| Service Request       | Lookup to Service Request | Related request.                                      |
| Comment Text          | Multiline text           | Comment entered by an employee or support agent.      |
| Comment Type          | Choice                   | Employee Comment, Internal Note.                      |
| Is Visible to Employee | Yes/No                  | Indicates whether the employee can view the comment.  |

The standard Dataverse Created By and Created On columns will identify the author and creation date. Internal notes must be protected through Dataverse security, not just hidden in the app.

### 2.10 Notification Log

**Logical name:** `eso_notificationlog`  
**Ownership:** Organisation owned

| Column              | Type                      | Notes                                                |
|---------------------|---------------------------|------------------------------------------------------|
| Notification Number | Autonumber                | Unique notification reference.                      |
| Service Request     | Lookup to Service Request | Related request, where applicable.                  |
| Recipient User      | Lookup to User            | Internal Dataverse user receiving the notification. |
| Notification Type   | Choice                    | Confirmation, Assignment, Approval, SLA, Closure.   |
| Channel             | Choice                    | Email, Teams.                                        |
| Status              | Choice                    | Pending, Sent, Failed.                               |
| Sent On             | Date and time             | Date and time notification was sent.                |
| Failure Details     | Multiline text            | Error details if notification fails.                |
| Correlation ID      | Text                      | Reference for tracing the related automation.       |

This table will help troubleshoot notification failures and prevent duplicate messages when automation runs are retried.

## 3. Table Relationships

| Parent Table       | Related Table       | Relationship |
|--------------------|---------------------|--------------|
| Request Category   | Service Request     | One-to-many  |
| Department         | Service Request     | One-to-many  |
| Service Request    | Request Assignment  | One-to-many  |
| Service Request    | Request Status History | One-to-many |
| Service Request    | SLA Tracking        | One-to-many  |
| SLA Configuration | SLA Tracking        | One-to-many  |
| Service Request    | Approval            | One-to-many  |
| Service Request    | Request Comment     | One-to-many  |
| Service Request    | Notification Log    | One-to-many  |

The solution will also use standard Dataverse tables:

- `systemuser` for requesters, agents, managers and approvers.
- `team` for support team assignments and routing.

## 4. Naming and Configuration Standards

- Use the publisher prefix `eso` for all custom tables and columns.
- Use global choices when the same set of values is shared across tables.
- Use reference tables for categories, departments and SLA configurations because these values need to be maintained as business data.
- Use alternate keys for Category Code, Department Code and SLA Code.
- Use Dataverse autonumber columns for request and transaction references.

## 5. Ownership and Security

The proposed ownership types are listed under each table. These must be confirmed before table creation because a Dataverse table's ownership type cannot be changed after creation.

Security will be configured through Dataverse security roles and teams. A lookup to a parent record does not automatically grant access to its related records.

The security design must cover:

- Employees accessing their own requests.
- Support agents accessing requests assigned to their team.
- Managers accessing relevant team and approval records.
- Internal notes remaining hidden from employees.
- Administrators maintaining reference and configuration data.

## 6. Implementation Order

Create the tables in dependency order:

1. Department
2. Request Category
3. SLA Configuration
4. Service Request
5. Request Assignment
6. Request Status History
7. SLA Tracking
8. Approval
9. Request Comment
10. Notification Log

After table creation, configure relationships, choices, alternate keys and views. Add sample data and validate the model before starting the Power Apps implementation.

## 7. Items to Validate

The following decisions will be finalised during implementation:

- Table ownership and record access strategy.
- Whether only one active assignment is allowed per request.
- Valid request status transitions and status history generation.
- Precedence of category-specific and priority-based SLA rules.
- SLA behaviour for paused and reopened requests.
- Approval stages and multiple approvers.
- Security for internal comments and operational history.
- Duplicate notification handling.

This document describes the proposed initial data model. It will be updated as the Dataverse schema is created and tested.
