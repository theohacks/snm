## **Practice Scenarios for ServiceNow Admin**

1**.Create a new user for a contractor, assign them to an "IT Support" group, and ensure they can only access the _Incident_ application.  
Solution:**

### **Create the Contractor User**

- Navigate to **Users** → _User Administration > Users_.
- Click **New**.
- Fill in details:
  - **User ID**: contractor1
  - **First name / Last name**: Contractor User
  - **Email**: <contractor1@gmail.com>
  - **Active**: Checked.
- Save.

### **Assign the User to the "IT Support" Group**

- On the user record, scroll to **Groups** (related list).
- Click **Edit**.
- Add to the **IT Support** group.
- Save.

### **Restrict Access to Only the Incident Application**

Now we need to make sure this contractor can only work with **Incident**.

#### **Option A: Role-Based Control (Mostly Preferred)**

- By default, Incident application requires **itil** role.
- Instead of giving full **itil** access (which gives too much), do the following:
  - Create a **new custom role**, ex: **incident_contractor**.
  - Assign this role only to permissions needed for Incident (using ACLs).
  - Assign the new role to your contractor user.
  - Do **not** give **itil** or other broad roles.

#### **Option B: Application Menu Restriction**

- Go to **System Definition > Application Menus**.
- Open the **Incident** application menu.
- In the **Roles** field, add your custom role (**incident_contractor).**
  - This ensures only users with this role can see the Incident.

### **Verify Access**

- **Impersonate** the contractor user.
- Check:
  - They should only see the **Incident application** in the left nav.
  - They can open/create/edit incidents (based on the ACLs you configured).
  - They cannot access other apps (like Change, Problem, etc.).

2\. **Assign a role to a new group so members can read _Knowledge Articles_ but cannot create or edit them.**

#### **Create a New Group**

- Navigate to User Administration > Groups.
- Click New.
- Enter a Name for the group (e.g., Knowledge Readers).
- Optionally, add a Description.
- Click Submit.

#### **Assign the Appropriate Role**

To allow read-only access to Knowledge Base articles, assign the **knowledge** role:

- Open the newly created group.
- Scroll to the Roles related list.
- Click Edit.
- Add the role: knowledge
  - This role allows users to view published articles.
- Click Save.

**\*\***Do NOT assign roles like **knowledge_admin or knowledge_manager,** which grant create/edit permissions.

#### **Add Users to the Group**

- In the group record, scroll to the Group Members related list.
- Click Edit.
- Select users you want to add.
- Click Save.

#### **Verify Access**

- Log in as one of the group members.
- Navigate to Knowledge > Articles.
- Confirm they can view articles.
- Try creating or editing an article - they should not have access.

3\. **Configure a UI Policy that hides the "Work Notes" field unless the state is "In Progress"**.  
**Solution:**

- **Navigate to UI Policies**
- Go to Application Navigator → type UI Policies → click System UI > UI Policies.
- Create a New UI Policy
- Click New.
- Select the Table → e.g., _Incident_ (or whichever table you're working on).
- Provide a Name (e.g., _Hide Work Notes unless In Progress_).
- In the Conditions section, set:
  - Field = _State_
  - Operator = _is_
  - Value = _In Progress_.
- Check the box Active.
- Save the record.
- **Add a UI Policy Action**
- In the same UI Policy record, scroll to UI Policy Actions (Related List).
- Click New.
- Configure the action:
  - Field name = _Work notes_
  - Visible = _True_ (since you want it visible only when the condition is met).
- Submit the action

4\. **Configure a UI Policy to hide Notes section in incident, when state is In Progress.**

**Solution**:

- **Navigate to UI Policies**
- Go to Application Navigator → type UI Policies → click System UI > UI Policies.
- Create a New UI Policy
- Click New.
- Select the Table → e.g., _Incident_ (or whichever table you're working on).
- Provide a Name (e.g., _Hide Work Notes unless In Progress_).
- In the Conditions section, set:
  - Field = _State_
  - Operator = _is_
  - Value = _In Progress_.
- Check the box Active.
- Save the record.
- **Make Run Script box True**
- Just write one line of code:
- g_form.setSectionDisplay('notes',false);
- Submit the action

5\. **Configure a response SLA, the SLA should pause, when the incident state is in On Hold vice versa**.

#### **Create or Modify an SLA Definition**

- Navigate to **Service Level Management > SLA Definitions**.
- Click **New** or open an existing SLA (e.g., "Response SLA").
- Fill in the basic details:
  - **Name**: Response SLA
  - **Table**: **Incident**
  - **Type**: Response
  - **Duration**: Set your desired time (e.g., 1 hour)

#### **Set SLA Conditions**

- Under the **Start Condition**:
  - Example: **State is New**
- Under the **Stop Condition**:
  - Example: State is Resolved or Closed
- Under the **Pause Condition**:
  - Add: **State is On Hold**

This ensures the SLA timer **pauses** when the incident is moved to **On Hold**, and **resumes** when it returns to another **New** state

#### **Test the SLA Behavior**

- Create a test incident.
- Confirm SLA starts when an incident is created.
- Change state to **On Hold** - SLA should pause.
- Change back to **Active** - SLA should resume.
- Resolve the incident - SLA should stop.

  6.**Configure an email notification that alerts the assigned group whenever a new _Change Request_ is created.**

**Solution:**

- **Navigate to Notifications**
- In the **Application Navigator**, type **Notifications**.
- Go to **System Notification > Email > Notifications**.

### **Create a New Notification**

- Click **New**.
- Fill in the basic details:
  - **Name**: _New Change Request Assigned Group Alert_
  - **Table**: _Change Request \[change_request\]_
  - **Active**: Checked

### **Define When to Send**

- Under **When to send**, configure:
  - **When to send**: _Insert_ (since you want this when a new record is created).

### **Define Who Will Receive**

- In the **Recipients** tab:
  - Under **Users/Groups in fields**, choose **Assigned to group** (or the field name for assigned group).
  - This ensures the entire assigned group gets the email.

### **Define What Will Contain**

- In the **What it will contain** tab:
  - **Subject**: New Change Request Created - \${number}

**Message HTML** (sample):  
<br/>A new Change Request has been created.

- - Number: \${number}
    - Short Description: \${short_description}
    - Requested By: \${requested_by}
    - Assignment Group: \${assignment_group}
    - State: \${state}

Please review and take necessary action.

### **Save & Test**

- Save the Notification.
- Create a new **Change Request** record, assign it to a group.
- Verify that the email goes out to all members of the Assigned Group.

7\. **Create a report showing the number of incidents opened by each department in the last 30 days.**

#### **Navigate to Reports**

- Go to Reports > Open Reports Modules.
- Click Create a Report.

#### **Define Report Source**

- Name: **Incidents by Department - Last 30 Days**
- Source Table: **Incident**

#### **Set Conditions**

- **Under Filter, add:**
  - Opened At → on or after → Today - 30 days
  - Department → is not empty **_(optional, to exclude unassigned)_**

#### **Choose Report Type**

- Select Type: **Bar Chart** or **Pie Chart** (or **List** if you prefer tabular view)

#### **Configure Grouping**

- Under Group By, select: **Department**
- Under Aggregation, choose: **Count**

#### **Save and Run**

- Click Save.
- Click Run to view the report.

8\. **Build a dashboard for Service Desk Managers showing KPIs like incidents by priority, created within a week, state wise also.**

#### **Step 1: Create Individual Reports**

**You'll need to create three separate reports first:**

#### **Incidents by Priority**

- Go to: Reports > Create New
- Name: Incidents by Priority
- Source Table: Incident
- Type: Bar Chart or Pie Chart
- Group By: Priority
- Aggregation: Count
- Filter: Opened At → on or after → Today - 30 days

#### **Incidents Created Within a Week**

- Name: Incidents Created - Last 7 Days
- Source Table: Incident
- Type: Time Series or Bar Chart
- Filter: Opened At → on or after → Today - 7 days
- Group By: Opened At (Daily)
- Aggregation: Count

#### **Incidents by State**

- Name: Incidents by State
- Source Table: Incident
- Type: Bar Chart or Pie Chart
- Group By: State
- Aggregation: Count
- Filter: Opened At → on or after → Today - 30 days

#### **Step 2: Create a Dashboard**

- Go to Self-Service > Dashboards.
- Click Create New Dashboard.
- Name: **Service Desk Manager KPIs**
- Add a Proper Description
- Click Submit.

#### **Step 3: Add Reports to the Dashboard**

- Open the newly created dashboard.
- Click Edit Content.
- Use Add Reports to include:
  - **Incidents by Priority**
  - **Incidents Created - Last 7 Days**
  - **Incidents by State**
- Arrange the widgets as needed for clarity.

9\. **Restrict the ability to delete records in the _Change Request_ table so only users with the "admin" role can do so.**

### **Navigate to Access Control (ACLs)**

- In the **Application Navigator**, type **Access Control**.
- Go to **System Security > Access Control (ACL)**.

### **Create a New ACL Rule**

- Click **New**.
- Fill in details:
  - **Type**: _record_
  - **Operation**: _delete_
  - **Table**: _Change Request \[change_request\]_
  - **Name**: _(auto-populates when you pick table + operation)_

### **Define the Condition / Role**

In the **Requires role** field, add: **admin**

- This ensures only users with the **admin** role can delete records.

### **Save & Test**

- Save the ACL.
- Test with a non-admin user → they should **not** see the delete option (or get a permission error if they try via URL).
- Test with an admin user → delete should work normally.

10\. **Create a custom table and create two reference fields (ex: assignment group and assigned to). Display the users based on selection of assignment group.**

### **Create a Custom Table**

- In the Application Navigator, type **Tables**.
- Go to **System Definition > Tables**.
- Click **New**.
  - Name: _u_custom_task_
  - Label: _Custom Task_.
  - Save.

### **Add Fields**

- Open your table and go to the **Columns** tab.
- Add two reference fields:
  - **Assignment Group** → Type = _Reference_, Table = _sys_user_group_.
  - **Assigned To** → Type = _Reference_, Table = _sys_user_.

### **Configure Reference Qualifier on "Assigned To"**

- We need to filter "Assigned To" users based on the selected Assignment Group.

#### **Using Reference Qualifier**

- Right click on the **Assigned To** field, click on **Configure Dictionary.**
- Go to **Dependent** Section, give the name of the Assignment Group(ex: u_ass_group)
- Update and Test the functionality.

  11.**How to auto assign incidents when user selects a category as network, the same incident be assigned to Network group.**

**Solution:**

- Go to Flow Designer → Designer.
- Click New Flow.
  - Name: Assign Incident by Category
  - Trigger: Created or Updated → Table = Incident
- Add a If action (Condition) with expression:
  - Select Trigger Record Category is Network
- Under the If branch, add Action → Update Record:
  - Record: Trigger → Incident(Trigger Record)
  - Set field Assignment group → Network
- Save and Activate the flow.
- Test the Flow.

**12\. HR Groups members are only able to see HR Related Records in servicenow?**

**Solution:**

### **Step 1: Create a Role for HR Access**

### Navigate to

User Administration → Roles → New

- Enter:
  - Name: hr_access
  - Description: Role to allow access to HR Cases
- Click Submit.

### **Step 2: Assign the Role to HR Group**

- Navigate to:  
   User Administration → Groups
- Open your HR group record.
- In the Roles tab → click Edit.
- Move hr_access from Available → Selected.
- Click Save.

Now all members of the HR group have the hr_access role.

### **Step 3: Create Access Control (ACL) for Viewing HR Cases**

- Navigate to:  
   System Security → Access Control (ACL)
- Click New.

Fill in:

| **Field** | **Value**          |
| --------- | ------------------ |
| Type      | record             |
| Operation | read               |
| Table     | Your HR Case table |
| Active    | True               |

### **Step 4: Define Access Condition (No Script)**

Scroll down to the Requires role section:

- Add the Role hr_access.

This means only users with the hr_access role can read/view HR Case records.

### **Step 5: Save and Test**

- Click Submit or Update to save the ACL.
- Impersonate a non-HR user:
  - Go to your profile → click Impersonate User → choose a user _not in the HR group_.
  - Try opening an HR Case record → You should see a "Security constraints prevent access to requested page" message.
- Now impersonate an HR group member:
  - They should be able to open HR Cases normally

**13\. When the Incident state changes to In Progress, Child incident related list should be hidden.**

**Solution:**

- Navigate to System UI → UI Policies → New.
- Fill the header:
  - Name: Hide related lists when State is In Progress
  - Table: Incident
  - Active: checked
  - Global: checked
- Condition: **State is In Progress**  
   (Use the exact label used in your instance for the In Progress state.)
- Submit the UI Policy record.
- In the UI Policy record click **New** under **UI Policy Actions**.

Set:

- **Field name:** select the related list-Child incident
- **Visible:** false
- **Read only:** optional
- Save and Test the UI Policy Action.

**14.How to Display Incident number while loading the incident form**

**Solution:**

- Navigate to System UI → Client Scripts → New.
- Fill the header:
  - Name: Show Incident Number on Load
  - Table: Incident
  - Type: onLoad
  - Active: True
- Add this script:

function onLoad() {

// Get the Incident number field value

var incNum = g_form.getValue('number'); // 'number' is the field name

alert('Incident Number: ' + incNum);

}

**15\. When the Incident state changes to In Progress, description should be hidden and short description should be mandatory.**

**Solution:**

## **Step 1 - Navigate to Client Scripts**

- Go to:  
   System UI → Client Scripts → New
- Fill the header:
  - Name: Hide Description and Make Short Description Mandatory
  - Table: Incident
  - Type: onChange
  - Field name: state
  - Active: checked

**Step 2 - Add the Client Script Code**

function onChange(control, oldValue, newValue, isLoading) {

if (isLoading) return;

if (newValue === '2') {

g_form.setDisplay('description', false);

g_form.setMandatory('short_description', true);

} else {

g_form.setDisplay('description', true);

g_form.setMandatory('short_description', false);

}

}

- Click **Submit** or **Update** to save.

**15**. **If the description field is empty in the incident table, prevent the form submission.**

**Solution:**

## **Step 1 - Navigate to Client Scripts**

- Go to:  
   System UI → Client Scripts → New
- Fill the header:
  - Name: Prevent Submit if Description Empty
  - Table: Incident
  - Type: onSubmit
  - Active: checked

**Step 2 - Add the Client Script Code**

function onSubmit() {

var description = g_form.getValue('description');

if (description == '') {

g_form.addErrorMessage('Description cannot be empty');

return false;

} else {

return true;

}

}

**16\. Users can not change the state field values in the incident list.**

**Solution:**

## **Step 1 - Navigate to Client Scripts**

- Go to:  
   System UI → Client Scripts → New
- Fill the header:
  - Name: Prevent State Inline Edit
  - Table: Incident
  - Type: onCellEdit
  - Field name: state
  - Active: checked

**Step 2 - Add the Client Script Code**

if(newValue==2){

alert('You can not edit this value');

saveAndClose==false;

}else{

saveAndClose==true;

}

**17\. How to set the Caller to Logged in user automatically in the incident table.**

**Solution:**

- Navigate: System Definition → Business Rules → New
- Fill the details:
  - Name: Set Caller on Incident Create
  - Table: Incident
  - When: before
  - Insert/update: checked
  - Advanced: true
- **Script**:

current.caller_id = gs.getUserID();

**18.When a user updates an incident record, priority should change to Critical automatically.**

**Solution:**

- Navigate: System Definition → Business Rules → New

- Settings:
  - Name: Set Priority field
  - Table: Incident
  - When: before
  - Update:checked
- Script:

current.impact = 1;

current.urgency = 1;

**19.Create a button on the Incident form that allows users to mark an Incident as Resolved with a single click.**

**Solution:**

- Navigate: System UI → UI Actions → New
- Settings:
  - Name: Resolve Incident
  - Table: Incident
  - Action type: Form button
  - Active: checked
- Script:
  - current.state = 6;
  - current.update();
  - action.setRedirectURL(current);

**20.Create a button on the incident table that copies the Short Description value into the Description field.**

**Solution:**

- Navigate: System UI → UI Actions → New
- Settings:
  - Name: Copy Short Description
  - Table: Incident
  - Action type: Form button
  - Active: checked

- Script:
  - current.description = current.short_description;
  - current.update();
  - action.setRedirectURL(current);

## **Viva Questions for ServiceNow Admin**

### **1\. What are Update Sets in ServiceNow?**

**Answer:  
**Update Sets are containers that capture configuration changes made in a ServiceNow instance. They allow you to move changes between instances (e.g., from development to test or production).

- - Always create and select your personal update set before making changes.
    - Do not include data records (like Incident records) in update sets.
    - Test update sets in sub-prod before moving to production.

### **2\. Explain the difference between ACLs and Roles in ServiceNow.**

**Answer:**

- **Roles**: Define what a user can do within the platform (e.g., itil, admin). They are permissions assigned to users or groups.
- **ACLs (Access Control Rules):** Define _record-level_ and _field-level_ access (read, write, create, delete). ACLs evaluate user roles, conditions, and scripts.  
   \*\*Roles are broad permissions; ACLs provide fine-grained security control.

### **3\. What is the difference between Client Script, and UI Policy?**

**Answer:**

- **Client Script:** Runs on the client (browser). Used for form field behavior (onChange, onLoad, onSubmit, onCellEdit). Example: Make a field mandatory on form load.

- **UI Policy:** Also client-side, but mainly used for dynamically changing field visibility, read-only state, or mandatory state without scripting.

### **4\. What is the difference between UI Policy and Data Policy?**

### **Answer:**

### **UI Policy**: Affects form fields only (client-side)

### **Data Policy**: Enforces rules across all interfaces (UI, imports)

### **5\. What is the Task Table in ServiceNow?**

### **Answer:**

### The Task table is a parent table in ServiceNow that provides common fields (like number, priority, state). Incident, Problem, Change, etc., extend from the Task table

### **6\. What are ServiceNow's different table types (Base, Extended, Core)?**

**Answer:**

- **Base Table:** Independent tables like Task \[task\].

- **Custom Table:** Created by the users/developers.

- **Core Table:** Provided by ServiceNow out of the box (like cmdb_ci, incident, change_request).  
   Inheritance allows consistency and reusability.

### **7\. What is the difference between Table.None, Table.\*, and Table.Field ACLs?**

**Answer:**

- **Table.None** → grants/denies access to the entire table .

- _Table._ \* → grants/denies access to all fields in the table

- **Table.Field** → grants/denies access to one specific field

### **8\. What happens if no ACL is defined for a table/field?**

**Answer:  
**If no ACL exists, ServiceNow **denies access by default**.  
Security is always "deny by default, allow by exception.

### **9\. what is the use of Impersonate user in servicenow**

### The _Impersonate User_ feature allows an administrator to log in as another user without needing that user's password

**Note:** Only admins or users with the "admin"role can impersonate.

### **10\. What is ServiceNow ?**

ServiceNow is a cloud-based platform designed to help organizations automate and manage their digital workflows across various departments like IT, HR, customer service, and security.

### **11\. What is a Service Catalog in ServiceNow?**

**Answer:  
**The Service Catalog is a collection of services, products, and offerings available to users within ServiceNow. It provides a user-friendly interface where employees can request items such as laptops, software, or access requests, and these are fulfilled through defined workflows.

### **12\. What is an Order Guide?**

**Answer**:

An Order Guide enables users to request multiple related items in a single submission. It streamlines the ordering process by grouping items and passing shared variables to each item if configured

### **13\. What is the use of Variables in Catalog Items?**

**Answer:  
**Variables are used to collect information from users when they submit a request. For example, in a "New Laptop Request," variables might include laptop model, RAM size, or additional software. Variables make the request form dynamic and customizable.

### **14\. What is a Record Producer in ServiceNow?**

**Answer:  
**A Record Producer is a type of catalog item used to create records in tables other than the Service Catalog tables. For example, a "Report an Issue" Record Producer can create an Incident record directly from the Service Catalog.

### **15\. What is the difference between a Catalog Item and a Record Producer?**

**Answer:**

- **Catalog Item:** Provides services or products (like hardware/software requests) and often follows a fulfillment workflow.

- **Record Producer:** Creates a record in a specific table (like an Incident, Change, or Problem) without needing a fulfillment workflow.

**16.What is a Variable Set and why is it useful?**

- **Answer:** A Variable Set is a reusable group of variables that can be applied to multiple Catalog Items. It promotes consistency and simplifies maintenance by allowing updates to shared variables in one place

### **17\. What is Flow Designer in ServiceNow?**

**Answer:  
**Flow Designer is a no-code/low-code tool used to automate processes in ServiceNow. It allows users to create, test, and execute workflows using natural language, triggers, and actions without writing complex scripts.

### **18\. What are the main components of Flow Designer?**

**Answer:  
**The main components are:

- **Flows** - The overall process definition.

- **Triggers** - Events that start the flow (e.g., record created/updated).

- **Actions** - Steps performed when the flow runs (e.g., create record, send email).

- **Data Pills** - Variables that hold and transfer data within the flow.

### **19\. What is the difference between Flow Designer and Workflow Editor?**

**Answer:**

- **Flow Designer** is modern, no-code, API-driven, and recommended for new automation.

- **Workflow Editor** is legacy and uses graphical drag-and-drop workflows. ServiceNow is moving towards Flow Designer as the standard.

### **20\. What are Subflows in Flow Designer?**

**Answer:  
**Subflows are reusable flows that can be called from other flows. They allow modular design, so common processes (e.g., sending notifications, logging tasks) can be built once and reused in multiple flows.

**21.What is an Action in Flow Designer?**

- **Answer:** An Action is a reusable unit that performs a specific task, such as creating a record, sending a notification, or calling an API. Actions can be reused across multiple flows

### **22\. What is an Import Set in ServiceNow?**

**Answer:  
**An **Import Set** in ServiceNow is a **staging table** used to import data from external sources (like Excel, CSV, or database) into ServiceNow.  
The imported data is first stored in the Import Set table and then moved to the target table (like Incident or User) using a Transform Map.

### **23\. What type of changes are captured in Update Sets?**

**Answer:  
**Update Sets capture most configuration changes such as table changes, form layouts, client scripts, business rules, workflows, and UI policies. They do not capture data (like records in the Incident table).

### **24\. What are the limitations of Update Sets?**

**Answer:  
**Some changes are **not captured** in Update Sets, such as:

- Data records (like incidents or users).
- Scheduled jobs.
- Some system properties.  
   In such cases, changes must be migrated manually.

### **25\. Why do we use Update Sets in ServiceNow?**

**Answer:  
**Update Sets are used to move customizations and configuration changes between different ServiceNow instances (Dev → Test → Prod) while keeping environments consistent.

### **26: What is Dot Walking in ServiceNow and how is it used?**

**Answer:  
**Dot Walking is a method in ServiceNow used to access fields on related tables by "walking" through reference fields. It allows you to retrieve data from a referenced record without writing complex scripts.

### **27: What is a Dictionary Override in ServiceNow and when do we use it?**

**Answer:  
**A **Dictionary Override** in ServiceNow allows you to change the dictionary settings of a field on a child table, without affecting the same field on the parent table.

### **28\. What is Knowledge Management in ServiceNow?**

**Answer:  
**Knowledge Management in ServiceNow is used to create, manage, and share articles that provide solutions, troubleshooting steps, FAQs, and best practices. It helps reduce repeated work for support teams and improves self-service for end users.

### **29\. What are Knowledge Bases in ServiceNow?**

**Answer:  
**A Knowledge Base is a collection of knowledge articles organized by topic or purpose. Different knowledge bases can be created for IT, HR, Facilities, or any business unit, each with its own access controls.

### **30\. How is access to Knowledge Articles controlled in ServiceNow?**

**Answer:  
**Access is managed through **User Criteria** and **Roles**. User Criteria can define who can view, contribute, or manage articles (for example, restricting HR knowledge to HR staff). This ensures only the right users see relevant content.

**31.What is an Application in ServiceNow?**

**Answer:  
**An **Application** in ServiceNow is a collection of related files, such as tables, forms, modules, workflows, and UI elements, that together deliver a specific business process or functionality.  
For example, the **Incident Management** application handles incident records and related activities.

**32.What is a Module in ServiceNow?**

**Answer:  
**A **Module** in ServiceNow is a **link or menu item** inside an application that opens a specific part of the application-such as a list of records, a form, a report, or a script.  
For example, under the _Incident_ application, modules include _Create New_, _All_, _Open_, and _Resolved_.

**33.What is the relationship between Applications and Modules?**

**Answer:  
**Modules exist **within** an application.  
An application is the parent container, and modules are the individual actions or views that let users interact with the application's data or functionality.

**34.Can you create custom applications and modules in ServiceNow? How?**

**Answer:  
**Yes.  
You can create a custom application using **Studio → Create Application**, and within that application, you can add modules using **Application Menu → New Module**.  
Custom applications are useful for building business-specific solutions.

**35.How can you control access to an application or module in ServiceNow?**

**Answer:  
**Access can be controlled using **roles**.  
You can assign specific roles to applications or modules so that only users with those roles can see or access them.  
For example, only users with the **itil** role can access the Incident Management modules.

**36.What is a Transform Map in ServiceNow?**

**Answer:  
**A **Transform Map** defines the **mapping between the Import Set table and the target table**.  
It tells ServiceNow how to transform the imported data - which fields in the Import Set correspond to which fields in the target table.

**37.What are the key stages in the data import process in ServiceNow?**

**Answer:  
**The data import process has three main stages:

- **Load Data** - Import external data into the Import Set table.

- **Create Transform Map** - Define how fields from Import Set map to the target table.

- **Run Transform** - Move and convert the data into the target table.

### **38.What is the purpose of a Coalesce field in a Transform Map?**

**Answer:  
**The **Coalesce field** is used to **find existing records** in the target table.  
If a match is found based on the coalesce field, the record is **updated**; if no match is found, a **new record** is created.  
It helps prevent duplicate records during imports.

**39.Can you run a Transform Map automatically after importing data?**

**Answer:  
**Yes.  
You can set the **"Run Transform"** option to automatically execute a Transform Map after the data load completes.  
Alternatively, you can manually run the transform from the **Import Set table record**.

**40**.**Can you use more than one field as a Coalesce field in a Transform Map in ServiceNow?**

**Answer:  
**Yes, You can select **multiple fields** as Coalesce fields in a Transform Map.  
<br/>If a record matches all coalesce fields, it will be **updated**; otherwise, a **new record** will be created.  
**41.What is a UI Action in ServiceNow?**

**Answer:  
**A **UI Action** in ServiceNow is a customization that adds buttons, links, or context menu items to forms and lists.  
It allows users to perform specific actions - such as saving a record, running a script, or redirecting to another page.

**42.What are the different types of UI Actions in ServiceNow?**

**Answer:  
**UI Actions can appear in different places based on their type:

- **Form button** - Appears as a button on the form.

- **Form context menu** - Appears in the "…" menu on a form.

- **List banner button** - Appears on the list header.

- **List context menu** - Appears when right-clicking a record in a list.

**43.Where can you create or configure a UI Action in ServiceNow?**

**Answer:  
**You can create a UI Action by navigating to:  
**System Definition → UI Actions → New**

**44.What is the purpose of the "Condition" field in a UI Action?**

**Answer:  
**The **Condition** field determines **when** the UI Action is visible or available.  
If the condition evaluates to **true**, the button or link will appear.

**45**.**Can UI Actions contain both client-side and server-side scripts?**

**Answer:  
**Yes, A UI Action can contain:

- **Client-side script** - Runs in the browser.
- **Server-side script** - Runs on the server.

**46.What is a Business Rule in ServiceNow?**

**Answer:  
**A **Business Rule** in ServiceNow is a **server-side script** that runs when a record is inserted, updated, deleted, displayed, or queried.  
It is used to automate processes, enforce data consistency, or perform actions based on record changes.

**47.What are the types of Business Rules in ServiceNow?**

**Answer:  
**We have four types of Business Rules:

- **Before** - runs before the record is saved to the database.
- **After** - runs after the record is saved.
- **Async** - runs asynchronously in the background.
- **Display** - runs before the record is sent to the client (used to pass data to client scripts).

**48.Where can you create or configure a Business Rule?**

**Answer:  
**You can create a Business Rule by navigating to:  
**System Definition → Business Rules → New  
**Then, specify the **Table**, **When to run (Before/After/Async/Display)**, **Conditions**, and **Script**.

**49.What is the difference between "Before" and "After" Business Rules?**

**Answer:**

- **Before Business Rule:** Runs before the record is saved. It can modify field values before inserting or updating the record in the database.
- **After Business Rule:** Runs after the record is saved. It's typically used for actions like sending notifications, creating related records, or triggering other processes.

**50.How can you prevent a record from being saved using a Business Rule?**

**Answer:  
**You can prevent a record from saving by using the **setAbortAction(true)** method in a Before Business Rule.
