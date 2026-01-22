# Rapid POS Emarsys Connector - Version 1.0
Updated 1/21/2026

---

## Overview

The Rapid Emarsys Connector automatically syncs customer and sales data from Counterpoint to Emarsys to support targeted email marketing. The connector syncs customer profiles and transaction information for customers with a populated **Email Address 1** in Counterpoint. 
  
Prior to considering this connector, please reach out to SAP Emarsys for a product quote.

---

## Minimum System Requirements

- Minimum Counterpoint version: **8.5.6.2**  
- Minimum SQL Server version: **2016**  
- Minimum Windows Server version: **2016**  
- Minimum PowerShell version: **5.1**  

If you would like the SAP Emarsys connector but your system does not meet these minimum requirements, please consult your Care Team Lead (vCIO) for an upgrade quote.

---

## Table of Contents

- [Minimum System Requirements](#minimum-system-requirements)
- [SECTION 1: Emarsys Customer Records](#section-1-emarsys-customer-records)
- [SECTION 2: Emarsys Configuration](#section-2-emarsys-configuration)
- [SECTION 3: Emarsys Account Store Mapping](#section-3-emarsys-account-store-mapping)
- [SECTION 4: Emarsys Field Mapping – Customers Up](#section-4-emarsys-field-mapping--customers-up)
- [SECTION 5: Emarsys Field Mapping – Document Headers Up](#section-5-emarsys-field-mapping--document-headers-up)
- [SECTION 6: Emarsys Field Mapping – Document Lines Up](#section-6-emarsys-field-mapping--document-lines-up)
- [SECTION 7: Emarsys Field Mapping – Document Payments Up](#section-7-emarsys-field-mapping--document-payments-up)
- [SECTION 8: Queued to Be Sent to Emarsys](#section-8-queued-to-be-sent-to-emarsys)
- [SECTION 9: Emarsys Customer Status View](#section-9-emarsys-customer-status-view)
- [SECTION 10: Mark All Emarsys Messages as Read](#section-10-mark-all-emarsys-messages-as-read)
- [SECTION 11: Emarsys Connector Execution and Sync Timing](#section-11-emarsys-connector-execution-and-sync-timing)
- [SECTION 12: Customer Profile Sync Logic and Workflow](#section-12-customer-profile-sync-logic-and-workflow)
- [SECTION 13: Managing Customer Email and Phone Updates](#section-13-managing-customer-email-and-phone-updates)
- [Conclusion](#conclusion)

---

## SECTION 1: Emarsys Customer Records

The Emarsys Connector adds an **Emarsys Customers** button within Counterpoint, providing access to Emarsys-specific customer fields directly from the Counterpoint customer record.

![Customer Record - Emarsys Customers Button](./images/counterpoint-customer-record-emarsys-customers-button.png)

The email address on the Emarsys customer record is populated from **Email Address 1** on the Counterpoint customer record.

![Emarsys Customer Record](./images/counterpoint-Emarsys-customer-record.png)

### Accessing Emarsys Customer Records

All Emarsys customer records can also be accessed from:

**Connectors > Emarsys > Emarsys Customer Records**

This view allows records to be displayed in **table view**, where filters can be applied to review customers based on their current sync status.

![Emarsys Customers in Table View](./images/counterpoint-Emarsys-customers-table-view.png)

### Emarsys Sync Status Codes

Each Emarsys customer record includes a sync status value indicating its current state in the sync process:

- **0** – Fully synced; nothing pending  
- **1** – Recently created or updated; will sync on the next connector run  
- **2** – Profile is currently in the active sync queue  
- **5** – Invalid email address  
- **9** – Sync error; requires remediation before it can be re-synced  

---

## SECTION 2: Emarsys Configuration

The Emarsys Connector includes a user interface for managing configuration options that control how the connector interacts with Emarsys and Counterpoint.  

![Emarsys Configuration](./images/counterpoint-Emarsys-configuration.png)

For clients who use **multiple Emarsys accounts**, a separate configuration record will exist for each account.

### Account Name
- Identifies the Emarsys account.
- Especially important for companies with more than one Emarsys account (for example, separate accounts for a retail store and a restaurant).

### Is Enabled?
- Used to temporarily disable the connector while troubleshooting or testing.

### Workgroup ID
- Workgroup **233** will be created when the connector is installed. This is not currently used, but rather is related to functionality that may be developed in the future. See "Import New Contact?" below.

### Auto Create Emarsys Profile
Controls whether or not Emarsys customer records are automatically created from Counterpoint.

- **Yes/Checked**  
  A Emarsys customer record is automatically created when a customer is added to Counterpoint with **Email Address 1**.
  - Customers pushed up to Emarsys are sent as "opt-in". 

- **NO**  
  Emarsys customer records must be created manually.

### Filter Customers Up
If this filter is populated, then only the customers that meet the requirements of the defind filter will be pushed up to Emarsys.

### Import New Contact?
PLACEHOLDER FOR FUTURE DEVELOPMENT. Currently the connector **cannot** create (insert) new Counterpoint customer records. It also cannot update existing customer records in Counterpoint.
- _When developed_, this field will control whether Emarsys contacts can create new Counterpoint customer records.
  - If set to yes/checked, Emarsys contacts that do not match an existing Counterpoint customer will be inserted into Counterpoint.
  - If set to no/unchecked, new Counterpoint customer records will not be created by the connector.

### Other Configuration Options
Additional configuration fields exist for internal use by Rapid programmers. These options are used to optimize performance or assist with troubleshooting and should not be modified by end users.

---

## SECTION 3: Emarsys Account Store Mapping

The Emarsys Connector includes functionality for pushing transactional data (documents) up to Emarsys. Only transactions from the stores configured in this mapping table will be pushed to Emarsys.
- Only customers with Emarsys Customer Records will have their transactions pushed to Emarsys.
- If a customer with a valid Emarsys Customer Record makes a purchase at a non-configured store, that particular transaction will not be pushed to Emarsys.

---

## SECTION 4: Emarsys Field Mapping – Customers Up

The **Emarsys Field Mapping – Customers Up** screen provides a user interface for managing which customer fields are sent from Counterpoint up to Emarsys.

This table defines how customer profile data in Counterpoint maps to Emarsys profile properties. The standard deployment includes a predefined set of fields that are automatically synced. Adjustments to this table should generally be performed by a programmer.

Note: This is best viewed in _table view_.

![Emarsys Field Mapping Customers Up in Table View](./images/counterpoint-Emarsys-field-mapping-customers-up-table-view.png)

Calculated fields are not included by default. Any request to add calculated fields must be reviewed and quoted separately by Rapid.

**Note:** **Email Address 1** is a required field and must be sent to Emarsys.

### Standard Customer Profile Fields Sent to Emarsys

The following customer fields are included in a standard Emarsys connector deployment:

1. Email Address 1 (Emarsys Field 3)
3. Customer Number
4. First Name (Emarsys Field 1)
5. Last Name (Emarsys Field 2)
6. Opt-in (Emarsys Field 31, default to 1, meaning yes)
7. Source (Emarsys Field 8874, defaults to Rapid)
8. Store ID (Emarsys Field 13887)

Additional fields can be mapped by request.

---

## SECTION 5: Emarsys Field Mapping – Document Headers Up

The **Emarsys Field Mapping – Document Headers Up** screen provides a user interface for managing which document header fields are sent from Counterpoint up to Emarsys.

This table defines how document header data in Counterpoint maps to Emarsys event fields. The standard deployment includes a predefined set of fields that are automatically synced. Adjustments to this table should generally be performed by a programmer.

Note: This is best viewed in _table view_.

![Emarsys Field Mapping Document Headers Up in Table View](./images/counterpoint-Emarsys-field-mapping-document-headers-up-table-view.png)

---

## SECTION 6: Emarsys Field Mapping – Document Lines Up

The **Emarsys Field Mapping – Document Lines Up** screen provides a user interface for managing which document line fields are sent from Counterpoint up to Emarsys.

This table defines how document line data in Counterpoint maps to Emarsys event fields. The standard deployment includes a predefined set of fields that are automatically synced. Adjustments to this table should generally be performed by a programmer.

Note: This is best viewed in _table view_.

![Emarsys Field Mapping Document Lines Up in Table View](./images/counterpoint-Emarsys-field-mapping-document-lines-up-table-view.png)

---

## SECTION 7: Emarsys Field Mapping – Document Payments Up

The **Emarsys Field Mapping – Document Payments Up** screen provides a user interface for managing which document payment fields are sent from Counterpoint up to Emarsys.

This table defines how document line payment in Counterpoint maps to Emarsys event fields. The standard deployment includes a predefined set of fields that are automatically synced. Adjustments to this table should generally be performed by a programmer.

Note: This is best viewed in _table view_.

![Emarsys Field Mapping Document Payments Up in Table View](./images/counterpoint-Emarsys-field-mapping-document-payments-up-table-view.png)


---

## SECTION 8: Queued to Be Sent to Emarsys

When sending transactional (document) data to Emarsys, each ticket is first placed into a queue in Counterpoint. Tickets are then synced from this queue to Emarsys during connector runs.

**Note:** Tickets are one type of document within Counterpoint. Currently this is the only document type pushed to Emarsys. (For example, orders and layaways are not sent to Emarsys.)

![Queued to be Sent to Emarsys](./images/counterpoint-Emarsys-queued-to-be-sent-to-Emarsys.png)

### Queue Status Values

Each document in the queue includes a status value indicating its current state in the sync process:

- **0** – Document has already been synced to Emarsys; nothing pending  
- **1** – Document has been recently created or updated and will be added to the queue on the next connector run  
- **2** – Document is currently in the active sync queue  
- **9** – Document encountered an error and requires remediation before it can be re-synced

---

## SECTION 9: Emarsys Customer Status View

Each Emarsys customer record includes a **sync status** that indicates its current state in the connector process. In some cases, it is helpful to review how many customer records fall into a particular status category.

For example, you may want to identify that **43 customers have an invalid email address (status 5)** so those records can be reviewed and corrected.

The **Emarsys Customer Status View** displays a summary table showing:
- Each sync status code (0, 1, 2, 5, 6, 9)
- The total number of customer records currently associated with that status

**Notes:**
- If no customer records exist for a given status, that status will **not** appear in the table.
- The table can be refreshed at any time to display the most up-to-date information.
- This is best viewed in _table view_.

![Emarsys Customer Status View](./images/Emarsys-customer-status-view.png)

For details on the meaning of each customer sync status value, refer back to **SECTION 1: Emarsys Customer Records**.

---

## SECTION 10: Mark All Emarsys Messages as Read

The **Mark All Emarsys Messages as Read** menu option allows users to suppress repeated pop-up alerts in Counterpoint while retaining all Emarsys connector messages for later review.

This is especially useful in scenarios such as:
- Repeated error messages following a temporary internet outage
- High-volume alert conditions that have already been reviewed or acknowledged

Marking messages as read stops the pop-up notifications but does **not** delete the messages. All connector messages remain accessible in Counterpoint and can be reviewed at any time.

---

## SECTION 11: Emarsys Connector Execution and Sync Timing

The Emarsys Connector operates as a **Windows Service**, automatically syncing customer profiles and transactional documents between Counterpoint and Emarsys.

The connector runs continuously in the background and is responsible for keeping both systems aligned while respecting Emarsys API rate limits and configured sync rules.

### Sync Intervals

The connector processes different types of data on separate schedules:

- **Customer Profiles**  
  New and updated customer profiles are synced every **15 minutes**.

- **Documents in the Queue**  
  Transactional documents are synced every **1 minute**.  
  This interval is configurable and may be adjusted to prevent Emarsys rate limiting.

If a document being synced contains a **new customer**, the customer profile is created in Emarsys **immediately as part of the document sync**. The connector does not wait for the next 15-minute customer profile sync cycle.

For details on how customer profile changes are evaluated and synchronized between Emarsys and Counterpoint, refer to **SECTION 13: Customer Profile Sync Logic and Workflow**.

---

## SECTION 12: Customer Profile Sync Logic and Workflow

This section describes the logical order and decision-making process used by the connector after a sync cycle begins.

The Emarsys Connector processes customer profile updates in a defined sequence to ensure that the most recent and authoritative data is preserved between Counterpoint and Emarsys.

### Step 1: Sync Changes from Emarsys Down to Counterpoint

The connector first retrieves profile changes made in Emarsys and evaluates whether those changes should be applied to Counterpoint.

- The connector compares the **date and time** of the most recent profile change in Emarsys to the **date and time** of the most recent update in Counterpoint.
- The system uses the values from the source with the **most recent timestamp**.

Behavior depends on configuration settings:

- If **Insert/Update Customers** is **enabled**, all configured fields are synced down from Emarsys to Counterpoint.
- If **Insert/Update Customers** is **disabled**, only **subscription status changes** are synced down.

### Step 2: Sync Changes from Counterpoint Up to Emarsys

After processing inbound changes, the connector identifies customer records in Counterpoint that have been **created or modified** since the previous sync.

These updates are then pushed up to Emarsys, ensuring that Emarsys profiles reflect the most current customer information stored in Counterpoint.

---

## SECTION 13: Managing Customer Email and Phone Updates

When a customer is synced to Emarsys, the connector stores the associated **Emarsys Profile ID** on the customer record in Counterpoint. This Profile ID becomes the permanent link between the Counterpoint customer and the Emarsys profile and is used for all future updates.  

Using the Profile ID ensures that customer history, engagement data, events, and flow activity are preserved in Emarsys even when identifying information changes.

### Updating Email Address and Phone Number

If **Email Address 1** or the configured phone number (**Mobile Phone 1** or **Phone 1**) is updated in Counterpoint for a customer who already has a Emarsys profile:

- The connector updates the email address or phone number on the **existing Emarsys Profile ID**.
- A new Emarsys profile is **not** created.
- The customer retains their full Emarsys history, including events, metrics, and flow participation.

This behavior ensures continuity in Emarsys while allowing customer contact information to be updated over time.

### Handling Duplicate Customer Records

The Emarsys Connector enforces strict rules to prevent **duplicate Emarsys profiles** and to maintain data integrity. Because Emarsys profiles are uniquely identified by email address (per Emarsys account), a single email address can only be associated with **one** Counterpoint customer record for that account.

The following scenarios describe how the connector behaves.

#### Scenario 1: Duplicate Email Addresses Already Exist in Counterpoint During Initial Setup

If the connector is installed and **multiple Counterpoint customers already share the same Email Address 1**:

- The connector creates or associates **one** Emarsys profile for that email address.
- Only one Counterpoint customer record can be linked to that Emarsys profile.
- Any additional Counterpoint customers using the same email address will **not** be able to create or associate their own Emarsys customer record for that email address.

This behavior is expected and prevents duplicate Emarsys profiles from being created during initial deployment.

#### Scenario 2: A Emarsys Customer Record Already Exists and the Same Email Is Assigned to Another Counterpoint Customer

If a Emarsys customer record already exists in Counterpoint for a given email address, and a user attempts to assign that **same Email Address 1** to a different Counterpoint customer record (either by editing an existing customer or creating a new one):

- Counterpoint blocks the action.
- An error is returned to the user.
- The connector does **not** allow a second Counterpoint customer to be linked to the same Emarsys profile.

This prevents multiple Counterpoint customer records from sharing a single Emarsys profile.

### Handling Merged Customers in Counterpoint

When two customer records are merged in Counterpoint:

- The Emarsys customer record associated with the **“To”** customer (the retained record) remains linked to the Emarsys profile.
- If the **“From”** customer had an associated Emarsys customer record, that record becomes detached from any active customer.

It is recommended to **manually delete** the detached Emarsys customer record after the merge. Otherwise, it will remain in Counterpoint with no functional association to an active Emarsys profile.

---

## Conclusion

The Rapid Emarsys Connector streamlines the exchange of customer profiles and transactional data between Counterpoint and Emarsys, enabling powerful email marketing.

For assistance with configuration changes, custom field mapping, or troubleshooting, contact Rapid Support.  
