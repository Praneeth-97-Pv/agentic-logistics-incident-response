Delivery Delay Automation – Business Case and Technical Solution

1. Problem Statement

UPS logistics operations rely on accurate, real-time coordination between multiple ServiceNow instances.
Each business unit (Logistics, Retail, and Delivery) operated in its own instance, creating data silos and communication delays.

When a delivery was late or exceeded its ETA window, three major issues occurred:

The delay was logged but not communicated to the target instance

Dispatch statuses were not automatically updated

Manual coordination created bottlenecks and SLA violations

These disconnects increased operational costs, created data inconsistencies, and resulted in delayed customer fulfillment updates.

2. Business Need

PepsiCo required an intelligent, automated solution that could:

Detect and calculate delivery delays in real time

Automatically propagate status updates between instances

Log financial impact (penalty rate) for late deliveries

Eliminate manual communication between teams

Enable proactive, data-driven dispatch management

The system needed to integrate predictive intelligence, automation, and cross-platform orchestration while remaining auditable and scalable.

3. Solution Overview

The final solution is an AI-enhanced ServiceNow orchestration framework that automates delay tracking and cross-instance updates using:

Multi-Client Processor (MCP) for secure instance-to-instance orchestration

Flow Designer for workflow sequencing and execution triggers

AI Agent Studio for intelligent categorization and adaptive routing

Custom ServiceNow Tables for structured delivery event storage

GlideRecord Scripts for CRUD operations and impact calculations

Now Assist / Predictive Intelligence for anomaly detection and pattern learning

When a route is delayed, the system calculates penalty impact, updates the Delivery Delay table, and notifies the connected instance to update the status to Dispatched.
An AI agent monitors MCP logs for anomalies and flags repeated failures for review.

4. Architecture Overview

The solution consists of three ServiceNow instances connected through MCP orchestration:

Instance

Role

Tool / Feature Used

Logistics

Triggers route execution

Flow Designer, MCP execute_route

Retail

Calculates ETA + penalty impact

Custom Script Action, Flow variables, AI Agent

Delivery

Updates status to "Dispatched"

GlideRecord logic, MCP update_execution_status

Support components include:

Custom Table: x_snc_pepsico_de_0_delivery_delay

AI Agent Studio Role: "Delivery Delay Analyzer"

Update Tool: update_execution_status

Integration User: Scoped with read/write and API access

Screenshot

Description

<img width="852" height="554" alt="Screenshot 2025-10-09 at 8 46 02 AM" src="https://github.com/user-attachments/assets/c17f8529-6eb6-4d76-b7d8-2fd76a138b85" />

End-to-End System Architecture – Full architecture view showing how the AI Agent, MCP tool, Script Include, and Flow Designer connect to automate delivery impact analysis.

<img width="752" height="345" alt="Screenshot 2025-10-09 at 9 18 20 AM" src="https://github.com/user-attachments/assets/aed38c71-d3b2-47e4-b0ba-f6c9ff572b73" />

Visual of Internal Issue – Demonstrates how mismatched route IDs and instance targeting caused data retrieval errors during initial MCP setup.

<img width="1458" height="770" alt="Screenshot 2025-10-09 at 9 12 51 AM" src="https://github.com/user-attachments/assets/cdd46e43-20fa-4284-a778-2246a1ae36b2" />

AI Agent Prompt – Shows the final AI Agent prompt configuration used to parse ETA, delivery window, and penalty rate inputs before calling the MCP tool.

<img width="1469" height="781" alt="Screenshot 2025-10-09 at 9 13 48 AM" src="https://github.com/user-attachments/assets/24355845-5c03-4798-ae68-24700f61d575" />

AI Agent Configuration – Displays the step-by-step AI Agent Studio configuration, including role statement, agent steps, and connected MCP reference.

<img width="1468" height="782" alt="Screenshot 2025-10-09 at 9 22 36 AM" src="https://github.com/user-attachments/assets/bc9b707f-3afc-456e-baad-501312e4f430" />

AI Agent Test Output – Displays the AI Agent execution results, including calculated impact, status update to “Dispatched,” and successful MCP-to-table record validation.

<img width="1467" height="773" alt="Screenshot 2025-10-09 at 9 25 33 AM" src="https://github.com/user-attachments/assets/9846bf14-7f3d-4d12-ba0a-4b59c51fef90" />

AI Agent Tools – Displays the configuration of tools connected to the Route Decision Agent within AI Agent Studio, including the MCP for route validation and the Script Include for automated decision logic execution.

5. Technology Stack

Category

Tool / Platform

Purpose

Platform

ServiceNow

Primary orchestration platform

Automation

Multi-Client Processor (MCP)

Secure cross-instance communication

Intelligence

AI Agent Studio / Predictive Intelligence

Smart detection and categorization of delivery issues

Workflow

AI Agent Studio

Trigger and manage automation steps

Scripting

GlideRecord (JavaScript)

Update and query Delivery Delay table

Analytics

Performance Analytics / Dashboard Widgets

Visualize dispatch metrics

Data Storage

Custom Scoped Table

Track delivery details, ETA, and penalties

6. Data Model and Logic

Field

Type

Description

route_id

Integer

Unique delivery route identifier

status

String

Current delivery state

eta_minutes

Integer

Time in minutes for ETA

stockout_penalty_rate

Decimal

Cost impact rate for delays

calculated_impact

Decimal

Computed penalty value

instance_name

String

Source instance of record

last_updated

Date/Time

Audit trail field for change tracking

Screenshot

Description

<img width="1457" height="805" alt="Screenshot 2025-10-09 at 9 03 06 AM" src="https://github.com/user-attachments/assets/ec464a3c-4527-46bf-92c0-ff4094e0dd5c" />

Delivery Delay Table – Displays stored logistics records containing ETA minutes, delivery window hours, penalty rates, and calculated impact values used by the AI Agent for delay analysis.

<img width="1456" height="804" alt="Screenshot 2025-10-09 at 9 02 33 AM" src="https://github.com/user-attachments/assets/d8f9729d-05fe-4075-b833-7440aa883699" />

Supply Agreement Table – Contains supplier, route, and contractual terms data referenced by the AI Agent during route validation and decision-making.

7. AI Agent Configuration

AI Agent Name: Delivery Delay Analyzer
Goal: Categorize and prioritize delay incidents based on ETA deviation, route frequency, and penalty rate.

Steps implemented in AI Agent Studio:

Retrieve incident details via MCP inputs

Analyze ETA deviation threshold

Assign a “Delay Severity” label using the trained model

Recommend escalation or auto-update path

<br>
<img width="1468" height="799" alt="Screenshot 2025-10-09 at 9 14 21 AM" src="https://github.com/user-attachments/assets/6906d90d-53bf-45de-a509-1b499c1ea6f1" />  – Orchestration workflow 

8. MCP Orchestration and AI Agent Logic

The orchestration was built in AI Agent Studio and controlled using MCP integration steps.

Step 1 – execute_route: Initiates the delivery route.

Step 2 – notify_delivery_delay: Calculates ETA variance and penalty rate.

Step 3 – update_execution_status: Updates the delivery record in the target instance.

Key Script Logic

(function(inputs) {
  var routeId = parseInt(inputs.route_id);
  var status = inputs.status || 'pending';
  
  var gr = new GlideRecord('x_snc_pepsico_de_0_delivery_delay');
  gr.addQuery('route_id', routeId);
  gr.query();
  if (gr.next()) {
    gr.status = status;
