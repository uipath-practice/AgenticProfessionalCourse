# Benefit Claims Processing

**Build an end-to-end agentic Benefit Claims process — BPMN modelling, IXP document extraction, three AI agents, human validation, and email notification.**

## Overview

Agentic orchestration, implemented in **Maestro**, lets you coordinate long-running enterprise
processes between humans, robots, and AI agents working across different systems of record and
systems of engagement. You design, implement, monitor, operate, and optimize those processes, then
run them at scale.

In this exercise you'll implement a **Benefit Claims** use case from the government sector — the
structured procedure citizens use to request financial or service-based assistance, such as
pensions, social assistance, or unemployment benefits.

| Step | Focus |
| ---: | :--- |
| [**Create BPMN process**](1-create-bpmn-process.md) | Model the end-to-end process on the BPMN canvas |
| [**Setting up the Solution**](2-setting-up-the-solution.md) | Create the agentic automation solution that implements the process |
| [**Configure a Robot**](3-configure-a-robot.md) | Add an RPA automation that uses IXP to read the application PDF |
| [**Residency Verification Agent**](4-residency-verification-agent.md) | Build an agent that checks the declared residency against known records |
| [**Income Verification Agent**](5-income-verification-agent.md) | Build an agent that checks the declared income against known records — generated using a coding agent and UiPath skills |
| [**Eligibility Determination Agent**](6-eligibility-determination-agent-coded.md) | Build a coded (Python) agent that decides eligibility using internal guidelines |
| [**Configure human validation**](7-configure-human-validation.md) | Present the agents' conclusions for a case worker to approve or deny |
| [**Configure API integration — Benefit Approval**](8-configure-api-integration-benefit-approval.md) | Notify the applicant when the claim is approved |
| [**Configure API integration — Benefit Rejection**](9-configure-api-integration-benefit-rejection.md) | Notify the applicant when the claim is rejected |
| [**Benefit Claims Assistant**](10-benefit-claims-assistant-conversational-agent.md) | Build a conversational agent that answers program questions and registers new applications from the chat |
| [**Publish the Solution**](11-publish-the-solution.md) | Publish, deploy and activate the solution, then run the process end to end |

!!! tip "Training Environment"
    Log in at **[cloud.uipath.com/tpenlabs](https://cloud.uipath.com/tpenlabs)** and remember using tenant **AgenticWorkshop** for this exercise.
