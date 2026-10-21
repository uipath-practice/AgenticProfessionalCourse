# Update external systems using API — Benefit Rejection Case

!!! tip "Here is our plan for this lesson:"

    1. Generate a benefits rejection email using Gen AI
    2. Send the email to the benefits applicant, using an **Integration Service** Gmail connector

## Goal

Let's configure the tasks in the workflow accordingly, so that the applicant is notified once the case
worker has rejected the claim.

## Integration Service

**UiPath** [Integration Service](https://docs.uipath.com/integration-service/automation-cloud/latest/user-guide/introduction)
is the fastest and most convenient way to automate API enabled applications. It takes care of
authorization and authentication, helping you centralize management of API connections and also
allowing faster integration into SaaS platforms.

In a nutshell, Integration Service does the following:

- Enables automation with an out-of-the-box library of connectors.
- Helps setting up and managing connections easily with standardized authentication.
- Enables kicking off automations with server-side triggers or events.
- Provides curated activities and events (with additional data filters) for ease of use.
- Simplifies automation design by providing a uniform experience across all Studio designers.

A [connection](https://docs.uipath.com/integration-service/automation-cloud/latest/user-guide/connections)
is an authenticated link to one application account, for example one Gmail mailbox. Your automations
use the connection to call that application, so nobody has to type credentials into a workflow.

Let's see what we have prepared:

![Orchestrator with the Benefit Claims folder selected, showing the available connections](9-configure-api-integration-benefit-rejection.images/1-integration-service-connections-W.png){ .screenshot width="900" }

There are 3 connections available:

- **Gmail Connection** that we can use to send emails
- **Data Fabric Connection** where we can store any type of data (tables, files, etc.)
- **GenAI Connection** which we will use to generate emails using LLMs

Good part: we don't need to worry about credentials or permissions today — platform administrators have
prepared these for our automation, and they are also taking care about securing and restricting access
to these.

!!! warning "Check your tenant"
    Make sure you are using the right tenant, or contact your trainer if there are issues with
    connections.

## Steps

### 1. Configure the Generate Rejection Email task

Let's configure the **Generate Rejection Email** task in Maestro. For this, we are going to use the
[UiPath GenAI Activities](https://docs.uipath.com/activities/other/latest/integration-service/uipath-uipath-airdk-about) —
specifically the
[Generate Email](https://docs.uipath.com/activities/other/latest/integration-service/uipath-airdk-airdk-generate-email)
activity.

- Action will be **Connector → Execute Connector Activity**
- Pick the **Generate Email** GenAI activity

![Activity picker searching for "generate email" with the Generate Email UiPath GenAI Activities result highlighted, and the Action set to Execute connector activity](9-configure-api-integration-benefit-rejection.images/2-select-connector-activity-W.png){ .screenshot width="900" }

[[[
![Generate Email activity properties panel](9-configure-api-integration-benefit-rejection.images/3-generate-email-properties.png){ .screenshot }
|50|
Next, let's configure the input properties listed below.
]]]

| Property | Value |
| :--- | :--- |
| Include salutation | **true** |
| Salutation | **Greetings** |
| Include sign-off | **true** |
| Sign-off | **Regards** |
| Sign-off name | **Benefits Claims Department** |
| Model | **gpt-5.1** |
| Salutation name | **Applicant** |
| Style | **Concise** |
| Output format | **HTML** |

Email content — enter this as a **JS Expression**:

```js
"Generate an email notifying the recipient that their application for benefits has been denied, for the following reason: "+vars.denialReason
```

### 2. Configure the Send Rejection Email task

Once we have the generated email, we can send it to the applicant using the Gmail connector. Let's
configure the **Send Rejection Email** task.

- Action will be **Connector → Execute Connector Activity**
- Pick the **Gmail - Send Email** activity

![Activity picker searching for "send email" with the Gmail Send Email result highlighted, and the Action set to Execute connector activity](9-configure-api-integration-benefit-rejection.images/4-select-gmail-activity-W.png){ .screenshot width="900" }

### 3. Configure the activity properties

- **To** — we are going to use the **in_EmailAddress** argument from the start event
- **Subject**:

```text
Benefits Application Rejected
```

![Gmail email activity with the uipathlabs@gmail.com connection selected](9-configure-api-integration-benefit-rejection.images/5-gmail-connection.png){ .screenshot }

- **Body** — the **emailContent** output variable from the **Generate Rejection Email** task

![Activity configuration panel with the To and Subject fields filled in](9-configure-api-integration-benefit-rejection.images/6-configure-email-properties.png){ .screenshot }

[[[
Pick the **emailContent** variable for the Body field.
|50|
![Variable picker showing the emailContent output variable](9-configure-api-integration-benefit-rejection.images/7-select-body-variable.png){ .screenshot }
]]]

### 4. Test the rejection scenario

Time to test the rejection scenario! Configure the debug values of the BPMN process as in the picture
below. In the **Case Worker Review** action center task, **reject** the benefits application.

In the debug panel, pass the following argument values:

```text
in_emailAddress:   your own email address
in_ExamplePaystub: Sample pay stub.png
in_Application:    Sample application.pdf
```

![Debug configuration dialog with the input arguments filled in](9-configure-api-integration-benefit-rejection.images/8-debug-arguments-W.png){ .screenshot width="900" }

Once the execution of the process is completed, you should receive the benefits rejection email:

![The received benefits rejection email opened in Outlook](9-configure-api-integration-benefit-rejection.images/9-rejection-email-W.png){ .screenshot width="900" }

**That completes the Benefit Claims Processing exercise.**
