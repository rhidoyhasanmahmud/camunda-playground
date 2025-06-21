1. Modeler

* A drag-and-drop tool (desktop or web) for creating **BPMN, DMN, and forms**.
* Supports building workflows, decision tables, connectors, and form-based user tasks.

2. Workflow & Decision Engine (Zeebe)

* The core engine that executes **processes and decisions**.
* Cloud-native, scalable, distributed (no central DB), resilient, handles microservices orchestration

3. Tasklist

* A ready-to-use **UI for human task management**.
* Displays tasks assigned to users, supports assignment, scheduling, and completion

4. Operate

* A monitoring tool for **real-time workflow visibility**.
* Inspect running instances, identify issues, retry or resolve failed tasks.

5. Optimize

* Provides **analytics and reporting** for continuous process improvement.
* Generates dashboards, finds bottlenecks, and helps in performance analysis.

6. Console

* The **admin interface** for platform management.
* Used for managing clusters, environments, users, roles, alerts, and API clients.

7. Forms

* **Camunda Forms** allow you to build input screens for user tasks with no custom frontend code.
* Built using the Modeler or JSON schema.
* Displayed in Tasklist for data entry, review, or approvals.

8. Connectors

* Prebuilt or custom **integrations** for calling APIs, sending emails, posting messages to systems like Slack, etc.
* Attached to Service Tasks in BPMN.
* Reduces the need for writing code to handle external communication.


| Component       | Role in the Process |
|----------------|---------------------|
| **Modeler**     | Used to **design** business processes (BPMN), decisions (DMN), forms, and connectors. |
| **Zeebe (Engine)** | Executes the **workflow and decisions**, manages task lifecycles, and handles orchestration. |
| **Tasklist**    | Provides a UI for **human users to view and complete tasks** (forms shown here). |
| **Operate**     | Enables real-time **monitoring, debugging, and management** of running process instances. |
| **Optimize**    | Used to **analyze and improve** process performance with dashboards and reports. |
| **Console**     | Central platform to **manage environments, roles, and credentials**. |
| **Forms**       | Provides **interactive screens** for data input during user tasks (e.g., approvals, requests). |
| **Connectors**  | Facilitate **integration with external systems** like REST APIs, email, Slack, databases, etc. |

🏦 Loan Approval Process — Component Roles

| Component        | Role in the Loan Approval Process |
|------------------|-----------------------------------|
| **Modeler**       | Design the loan workflow: <br> • **Start** → Submit Form → Check Credit (API) → Review → Decision → Notify |
| **Zeebe (Engine)**| Executes the entire workflow and decision table (DMN), manages task transitions and data flow |
| **Tasklist**      | Allows the **loan officer** to open and complete the "Review Application" task via a form |
| **Operate**       | Enables team to **monitor loan application status**, retry failed tasks, and trace workflow problems |
| **Optimize**      | Generates insights like: <br> • Avg. time to approve loans <br> • Number of rejections <br> • Bottlenecks |
| **Console**       | Manage platform setup: <br> • Add loan officers as users <br> • Configure API clients and environments |
| **Forms**         | Used to collect applicant data (name, income, amount) and let officer input decision/comments |
| **Connectors**    | Used for: <br> • Calling a credit check API <br> • Sending approval/rejection emails |


📋 Flow Summary (Step-by-Step):

1. Customer fills form → Camunda Form
2. System checks credit → HTTP Connector
3. Officer reviews request → Tasklist + Form
4. DMN decision table evaluates credit score
5. Email sent → Email Connector
6. All activities monitored via Operate, Optimize, and managed via Console