**What is a Cluster?**

A cluster is a group of computers (called nodes) working together as one system to:

* Share the work
* Handle more load
* Stay running even if one fails

In Camunda 8, a Cluster Means: A **group of components (Zeebe brokers, gateways, monitoring tools)** running on multiple
servers (or containers) that **together handle your business workflows**.

**Why is Clustering Important?**

| Without Cluster       | With Cluster                         |
|-----------------------|--------------------------------------|
| Runs on 1 server      | Runs across multiple servers         |
| Risk of full downtime | No downtime even if one server fails |
| Limited performance   | Can handle thousands of workflows    |
| Hard to scale         | Easy to scale by adding nodes        |

**Cluster Types (SaaS)**

| Cluster Type | Best For                          | Uptime Promise (Orchestration Cluster*) |
|--------------|-----------------------------------|-----------------------------------------|
| Basic        | Dev/testing, experiments          | ~99% uptime                             |
| Standard     | Production environments           | ~99.5% uptime                           |
| Advanced     | Mission-critical, minimal outages | ~99.9% uptime                           |

**Cluster Size**

* Sizes: 1×, 2×, 3×, 4×, based on performance needs.
* Bigger = more capacity for concurrent workflows and data retention
* You can resize anytime (up or down). For sizes over 4×, contact support

**So, Why Cluster?**

If you're building anything production-grade, like:

* Banking apps
* Insurance claim systems
* eCommerce order processing
* Government approvals

...you need:

* High availability
* Horizontal scalability
* Fault tolerance

A Cluster is the answer.

---

**What is Camunda Gateway?**

The Camunda Gateway is the entry point into the Zeebe Cluster (Camunda 8 workflow engine).

* 🟢 It accepts client requests (e.g., deploy workflow, start instance)
* 🟡 Then routes those requests to the correct Zeebe broker/partition.

Think of It Like:

A reception desk at a big office:

* You (client) go to the front desk
* The receptionist (Gateway) checks where your request should go
* Then sends it to the right department (Zeebe Broker)

Gateway Key Features

| Feature            | Description                                          |
|--------------------|------------------------------------------------------|
| Stateless          | Doesn’t store process data                           |
| Scalable           | You can run multiple gateways behind a load balancer |
| gRPC Communication | Talks to Zeebe brokers via gRPC                      |
| Secure Front Door  | You can enforce authentication and TLS here          |

```text
[Client / App] ──► [Camunda Gateway] ──► [Zeebe Broker(s)]
                            │
                   (routes to correct partition leader)
```

So

- Entry point for all client interactions with Camunda
- It knows how to route requests to the correct broker
- It not store data because it stateless

---

**What is a Zeebe Broker?**

A Zeebe Broker is the core engine in Camunda 8 that:

* Executes workflows
* Stores workflow state
* Handles service tasks, timers, gateways, etc.

A Zeebe Broker is the “brain” inside a Camunda cluster that does all the actual work for your BPMN diagrams.

If Camunda is like a factory:

* Modeler = Blueprint designer 📝
* Gateway = Front desk 🛎️
* Zeebe Broker = Factory worker 👷 (executes the steps)

**How Brokers Work Together in a Cluster**

Imagine you have 3 brokers in your Camunda 8 cluster:

```text
[Broker A] ← Leader of Partition 1
[Broker B] ← Leader of Partition 2
[Broker C] ← Leader of Partition 3
```

A Camunda 8 Cluster splits workflow data into partitions (like work zones).

Each partition is handled by:

* 1 Leader Broker (active, executes workflows)
* 2+ Follower Brokers (passive, backup copy)

This is how Zeebe achieves scalability and fault-tolerance.

Imagine you're managing 3 pizza branches (brokers) and your city has 3 delivery zones (partitions):

| Zone (Partition) | Main Branch (Leader) | Backup Branches (Followers) |
|------------------|----------------------|-----------------------------|
| Zone 1           | Branch A             | B, C                        |
| Zone 2           | Branch B             | A, C                        |
| Zone 3           | Branch C             | A, B                        |

If it's the Leader of a partition:

* It executes tasks in that zone
* Accepts and processes workflow instances
* Sends jobs (e.g., to your app) and moves process forward

If it's a Follower of a partition:

* It keeps a copy of the data
* Does not execute tasks
* If the leader crashes, it can become the new leader

```text
       Partition 1   Partition 2   Partition 3
       -----------   -----------   -----------
Leader    Broker A      Broker B      Broker C
Follower  Broker B,C    Broker A,C    Broker A,B
```

---
**Start with the Analogy: School Exams**

Imagine a school is conducting exams for 300 students.

But instead of putting everyone in one big hall, they:

- Divide the students into 3 classrooms (to balance load)
- Assign 1 teacher per classroom to manage the exam

Now think:

- Each classroom = Partition
- Each teacher = Broker

**Now Let’s Map It to Camunda**

A partition is a slice of data and responsibility.

* If you have 3 partitions, Camunda divides workflow instances among them.
* Each instance runs only within one partition (never hops between them).

A Zeebe Broker is a worker node in the cluster.

* It can lead some partitions and follow others.
* It executes workflows only if it's the leader of that partition.

**Summary**

| Term      | Meaning                                                |
|-----------|--------------------------------------------------------|
| Partition | A unit of data & task ownership for workflows          |
| Broker    | A node in the cluster that manages partitions          |
| Leader    | The broker actively executing workflow for a partition |
| Follower  | Brokers that back up the partition data                |

---

**A Overview**

`Step 1: Spring Boot Loads BPMN`

On app startup, you deploy the workflow like this:

```java
@Autowired
private ZeebeClient zeebeClient;

@PostConstruct
public void deployWorkflow() {
    zeebeClient.newDeployResourceCommand()
        .addResourceFile("bpmn/loan-approval.bpmn")
        .send()
        .join();
}
```
The BPMN file from resources/bpmn/ is read and sent to Camunda Gateway.

`Step 2: Gateway Routes to a Broker`

- Camunda Gateway receives the deployment request
- It picks the right partition leader (e.g., P1 → Broker A)
- That broker stores the process definition and replicates it to others

🟢 Workflow is now live in the Camunda cluster!

`Step 3: Your App Starts the Workflow`

Whenever needed (e.g., user submits form), you start a new workflow instance:

```java
zeebeClient.newCreateInstanceCommand()
    .bpmnProcessId("loan_approval")
    .latestVersion()
    .variables(Map.of("applicant", "John", "amount", 7000))
    .send()
    .join();
```
📤 This request again goes to the Gateway, and is routed to the leader of a partition (e.g., P2 → Broker B)

`Step 4: Camunda Creates Jobs`

Let’s say your BPMN includes a Service Task like "check-credit":

```xml
<bpmn:serviceTask id="checkCredit" name="Check Credit" zeebe:taskDefinitionType="check-credit" />
```
Camunda creates a job of type check-credit.

`Step 5: Spring Boot Handles the Job`

You run a job worker in your app:

```java
@PostConstruct
public void registerWorker() {
    zeebeClient.newWorker()
        .jobType("check-credit")
        .handler((client, job) -> {
            Map<String, Object> vars = job.getVariablesAsMap();
            boolean approved = (int) vars.get("amount") < 10000;

            client.newCompleteCommand(job.getKey())
                .variables(Map.of("approved", approved))
                .send()
                .join();
        })
        .open();
}
```
📥 Your app receives the task, processes it, and completes the job back to Zeebe.

`Step 6: Workflow Progresses`

* Workflow goes to next step: maybe a user task or decision
* If there’s a user task, it shows up in Camunda Tasklist
* Admins can monitor progress in Operate
* Business users can see analytics in Optimize

```text
[Spring Boot App on ECS] 
     |
     |-- Deploy BPMN → [Gateway] → [Zeebe Broker (Partition Leader)]
     |
     |-- Start Workflow → [Partition] → Creates Job
     |
     |-- Job Worker picks up job → Completes it
     |
     |-- Workflow continues → Tasklist / Operate / Optimize
```




