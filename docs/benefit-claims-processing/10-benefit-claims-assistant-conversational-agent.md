# Build the Benefit Claims Assistant (Conversational Agent)

!!! tip "Here is our plan for this lesson:"

    1. Build a **conversational** AI Agent in **Studio Web** that answers questions about the benefits program, using the **Eligibility** context
    2. Let the same agent register new applications: collect the documents in the chat, upload them and start the **Benefit Claims Processing** workflow
    3. Test both scenarios in the agent's chat

## Goal

By the end of this lesson you'll have a **Benefits Assistant** that applicants can chat with. It answers questions about the SNAP program from the official documents, and it registers a new application end to end: it asks for the applicant's email, then the application, then the paystub, stores both files and starts the existing **Benefit Claims Processing** workflow, without waiting for the decision.

## What we are building

The agent has two jobs:

- **Job A: answer questions** about the SNAP benefits program, using only the official program documents in the `Eligibility` index.
- **Job B: register a new application.** It collects the applicant's email address, then the application, then the paystub, uploads each file, and starts the existing **Benefit Claims Processing** workflow.

```text
                        Applicant (chat window)
                                  |
                                  v
              +-----------------------------------------+
              | Benefits Assistant Conversational Agent |
              +-----------------------------------------+
                 |                 |                  |
     Job A       |       Job B     |                  |   Job B (after "yes")
                 v                 v                  v
        +--------------+  +---------------------+  +----------------------+
        | Eligibility  |  | UploadFileTo        |  | Start Benefit Claim  |
        | index (RAG)  |  | StorageBucket  (x2) |  | (robot, no waiting)  |
        +--------------+  +---------------------+  +----------------------+
                                   |                          |
                                   v                          v
                        "Input Documents" bucket   BenefitClaimsProcessing (BPMN)
                        application + paystub      agents, case worker review,
                                                   approval / rejection email
```

| Piece | Type | What it does | Why it is there |
|---|---|---|---|
| **Benefits Assistant Conversational Agent** | Low-code agent, **Conversational** | Chats with the applicant, answers questions, runs the registration flow | One friendly front door for applicants |
| **Eligibility** | Context Grounding index (already in the solution) | Returns the most relevant passages from the program documents | The agent answers from official rules, not from its own memory |
| **UploadFileToStorageBucket** | RPA robot (already in the template) | Saves a file the user attached in the chat into the **Input Documents** bucket and returns the real file name | The BPMN reads its input files from that bucket. A robot with a file argument also lets the chat accept file uploads |
| **Start Benefit Claim** | RPA robot (already in the template) | Starts the `BenefitClaimsProcessing` process with the two file names and the email, then returns right away | No Integration Service connector needed. It does not wait, because the BPMN can run for days |
| **BenefitClaimsProcessing** | BPMN agentic process (already in the solution) | Extracts data, runs the verification agents, sends the case to a case worker, emails the result | Unchanged. The chat simply becomes a new way to start it |

!!! info "Key idea"
    The agent decides *what* to do and *when*. Robots do the parts that must be exact: storing files, stamping the real time, starting a process. An LLM has no reliable clock and should never guess a timestamp.

## Conversational vs. autonomous agents

An **autonomous** agent, like the ones you built in the earlier lessons, takes one input and returns one structured output. A **conversational** agent talks with a person over many turns, streams its replies and can ask follow-up questions. Registration needs exactly that.

That's also why a conversational agent looks different:

- **No output arguments.** A conversational agent's "output" is the chat itself: its messages and the tool calls it makes. Nothing would ever fill an output schema, so leave it empty.
- **The user prompt stays blank.** In an autonomous agent, the user prompt turns input arguments into a task. Here, **each chat message from the user is the input**. The platform sends the whole conversation to the agent on every turn.

You also don't add any input argument for "message" or "chat history". The platform provides the conversation automatically.

## Steps

### 1. Create the conversational agent

1. In the Explorer, click **Add** (**+**) and choose **Agent** under **Building Blocks**.

    ![Studio Web Explorer with the add (+) button and the Agent option under Building Blocks highlighted](10-benefit-claims-assistant-conversational-agent.images/1-add-agent-to-solution.png){ .screenshot }

2. Name the agent:

    ```text
    Benefits Assistant Conversational Agent
    ```

3. In the **Select agent type** dialog, choose **Conversational** under **LOW-CODE** and click **Start fresh**.

    ![Select agent type dialog with the Conversational option selected under LOW-CODE and the Start fresh button highlighted](10-benefit-claims-assistant-conversational-agent.images/2-select-conversational-agent-type.png){ .screenshot }

4. In the agent settings, choose the model `anthropic.claude-sonnet-5` (Claude Sonnet 5).

    ![Benefits Assistant Conversational Agent selected on the canvas, with the agent settings panel showing the anthropic.claude-sonnet-5 model highlighted](10-benefit-claims-assistant-conversational-agent.images/3-select-model-W.png){ .screenshot width="900" }

The agent now appears in the Explorer, its type is **Conversational**, and it has no input or output arguments.

### 2. Add the Eligibility context

[Context Grounding](https://docs.uipath.com/automation-cloud/automation-cloud/latest/admin-guide/about-context-grounding)
is a component of the UiPath AI Trust Layer which allows you to bring in your data to generate more
accurate, reliable GenAI predictions. It stores your documents in an
[index](https://docs.uipath.com/orchestrator/automation-cloud/latest/user-guide/about-indexes): a
searchable copy that returns the passages that best match a question.

The index contains the program documents (eligibility criteria and benefit calculation rules). With it, the agent answers from the official text and can show where the answer came from. Without it, the agent would answer from general knowledge, which may be outdated or simply wrong for this program.

1. In the agent, click **Add Context**.

    ![Agent canvas with the add (+) button under Context highlighted](10-benefit-claims-assistant-conversational-agent.images/4-add-context.png){ .screenshot }

2. Select **Context Grounding Indexes**.

    ![New context dialog with Context Grounding Indexes highlighted above Data Fabric Entities](10-benefit-claims-assistant-conversational-agent.images/5-select-context-type.png){ .screenshot }

3. Select the **Eligibility** index from the current solution.

    ![Indexes picker with Eligibility highlighted under the Benefit Claims folder in Available resources](10-benefit-claims-assistant-conversational-agent.images/6-select-eligibility-index.png){ .screenshot }

4. Choose the **Semantic** search strategy.

    ![Strategy dropdown with the Semantic option highlighted](10-benefit-claims-assistant-conversational-agent.images/7-select-semantic-strategy.png){ .screenshot }

5. Paste this as the context **description**:

    ```text
    SNAP program eligibility rules: income limits, deductions, household size rules and benefit amounts.
    ```

    ![Eligibility context settings with the Description field filled in and highlighted](10-benefit-claims-assistant-conversational-agent.images/8-context-description.png){ .screenshot }

6. Keep the search query **dynamic** (the agent writes the query) and paste this as the query description:

    ```text
    A focused natural-language question about SNAP eligibility rules, income limits, deductions or benefit amounts.
    ```

    ![Eligibility context settings with the dynamic Query field filled in and highlighted](10-benefit-claims-assistant-conversational-agent.images/9-context-query.png){ .screenshot }

### 3. Add the UploadFileToStorageBucket tool

This robot stores each attached file in the **Input Documents** bucket. It also has a **file** argument (`In_File`): a tool that can process files is what enables file uploads in the chat. Without it, users can't attach files.

1. In the tools section, click **+**.

    ![Agent canvas with the Eligibility context attached and the add (+) button under Tools highlighted](10-benefit-claims-assistant-conversational-agent.images/10-add-tool.png){ .screenshot }

2. Search for **UploadFileToStorageBucket** under RPA workflows and select it.

    ![Choose tool panel with UploadFile typed in the search box and UploadFileToStorageBucket highlighted under RPA workflows in solution_folder](10-benefit-claims-assistant-conversational-agent.images/11-select-upload-tool.png){ .screenshot }

3. Paste this into the tool **Description**. It tells the agent what the tool does and when to call it. The details of each value go in the argument descriptions, in the next step.

    ```text
    Uploads ONE file the user attached in the chat to the 'Input Documents' storage bucket and returns the exact name it was stored under. Call it once per document, one at a time: first for the application, then, after the user has attached it, for the paystub.
    ```

4. Fill in the argument descriptions (the text the agent reads to decide what to pass):

    - `In_File`:

        ```text
        The single file the user attached for the current document: the application (PDF) or the paystub (PDF or PNG)
        ```

    - `in_emailAddress`:

        ```text
        Applicant email address, exactly as provided by the user
        ```

    - `in_FileName`:

        ```text
        Target file name: <sanitized email>_<application|paystub>_{timestamp}.<pdf|png>. Sanitized email = the email with every '@' and '.' replaced by '_'. Keep the literal text {timestamp}, the robot replaces it. The application is always .pdf; the paystub extension must match the attached file. Example: jane_doe_example_com_paystub_{timestamp}.png
        ```

![UploadFileToStorageBucket tool settings with the Description and the In_File, in_emailAddress and in_FileName argument descriptions filled in and highlighted](10-benefit-claims-assistant-conversational-agent.images/12-upload-tool-configuration.png){ .screenshot }

### 4. Add the Start Benefit Claim tool

Let's add the tool that triggers the Maestro process for claim submission.

1. In the tools section, click **+**.

    ![Agent canvas with the add (+) button under Tools highlighted](10-benefit-claims-assistant-conversational-agent.images/10-add-tool.png){ .screenshot }

2. Search for **Start Benefit Claim** and select it.

    ![Choose tool panel with Start Benefit Claim typed in the search box and the Start Benefit Claim workflow highlighted under RPA workflows in solution_folder](10-benefit-claims-assistant-conversational-agent.images/13-select-start-benefit-claim-tool.png){ .screenshot }

3. Paste this into the tool **Description**:

    ```text
    Registers and starts a new SNAP benefit claim by launching the Benefit Claims Processing workflow in the background. Call it only once per application, after both documents are uploaded and the user has explicitly confirmed the summary. It does not return an eligibility outcome.
    ```

4. Fill in the argument descriptions:

    - `in_Application`:

        ```text
        The out_FileName returned by UploadFileToStorageBucket for the application PDF, exactly as returned
        ```

    - `in_ExamplePaystub`:

        ```text
        The out_FileName returned by UploadFileToStorageBucket for the paystub (PDF or PNG), exactly as returned
        ```

    - `in_emailAddress`:

        ```text
        Applicant email address, the same one used for the uploads
        ```

    ![Start Benefit Claim tool settings with the Description and the in_Application, in_ExamplePaystub and in_emailAddress argument descriptions filled in and highlighted](10-benefit-claims-assistant-conversational-agent.images/14-start-benefit-claim-tool-configuration.png){ .screenshot }

### 5. Write the System Prompt

The system prompt is the agent's job description. It sets the scope, the two jobs, the exact registration steps and the rules for tools. A vague prompt is the most common reason an agent misbehaves.

The prompt covers:

- **Job A, answer program questions:** only from the Eligibility context; at most 3 searches per question; if the documents don't contain the answer, say so; never decide whether a specific person is eligible or estimate their benefit (that's the case worker's job, through the BPMN).
- **Job B, register an application, one file at a time:**
    1. Ask for the email address.
    2. Ask for the **application** (PDF, max 5 MB), upload it as `<sanitized email>_application_{timestamp}.pdf`, and confirm it was received.
    3. Only then ask for the **paystub** (PDF or PNG, max 5 MB) and upload it as `<sanitized email>_paystub_{timestamp}.pdf` or `.png`, matching the file.
    4. If the user attaches two files at once, ask them to send one at a time, starting with the application.
    5. Summarise (email + the two stored file names) and ask for confirmation.
    6. Call **Start Benefit Claim** with the two returned `out_FileName` values and the email.
    7. Tell the user the application was submitted and they'll be contacted by email.

The "sanitized email" replaces every `@` and `.` with `_`. Example: for `jane.doe@example.com` the agent sends `jane_doe_example_com_application_{timestamp}.pdf`, and the robot stores it as, for example, `jane_doe_example_com_application_20261016143005.pdf`.

Open the agent, select the **System prompt** field and paste the prompt below. Leave the **User prompt** empty.

```text
You are the Benefits Assistant for the SNAP benefits program. You help members of the public understand the program rules and register a new benefit application. Tone: friendly, patient, clear and concise. Use plain language and avoid jargon.

Scope:
- In scope (Job A): questions about SNAP eligibility rules, deductions, income limits, household size rules and benefit amounts.
- In scope (Job B): registering a new benefit application by collecting the applicant's email, then the application (PDF), then the paystub (PDF or PNG), one at a time, and submitting the claim.
- Out of scope: any other topic, legal or tax advice, the status or outcome of an existing claim, and changes to a submitted claim. Politely say you cannot help with that and, where relevant, point the user to the benefits office.

Conversation behavior:
- On the first turn, greet the user briefly and explain that you can answer questions about SNAP eligibility rules or help them submit a new application.
- If the user's intent is unclear, ask one short clarifying question before acting.
- Handle one job at a time. If the user switches from questions to applying (or back), follow the new request.

Job A: answering eligibility questions
- Answer ONLY from information retrieved from the Eligibility context. Do not use outside knowledge about SNAP or other programs.
- Search the Eligibility context with a focused query. Search at most 3 times per user question. After the third search, stop searching and answer with what you already have.
- If the retrieved content does not cover the question, or only covers part of it, say so clearly and do not guess.
- Explain rules in general terms. Never make an eligibility decision for a specific person, never say that someone qualifies or does not qualify, and never promise or estimate a benefit amount for a specific person or household. If the user asks whether they personally qualify or how much they will get, explain the relevant general rules and invite them to submit an application so the program can assess their case.

Job B: registering a new application
Follow these steps in order. Do not skip steps.
1. Ask for the applicant's email address. If the value does not look like a valid email address, ask again.
2. Collect and upload the application first. Ask the user to attach ONLY the completed application form (a PDF file, at most 5 MB) and wait for it. Treat the file attached in reply to this request as the application. If it is missing, is not a PDF, or is larger than 5 MB, explain the problem and ask for a correct file. Then upload it with the UploadFileToStorageBucket tool: In_File = the attached application, in_emailAddress = the email address, in_FileName = <sanitized email>_application_{timestamp}.pdf. Keep the returned out_FileName and tell the user the application was received.
3. Then collect and upload the paystub. Only after the application upload has succeeded, ask the user to attach ONLY a recent paystub (a PDF or a PNG image, at most 5 MB) and wait for it. Treat the file attached in reply to this request as the paystub. If it is missing, has another format, or is larger than 5 MB, explain the problem and ask for a correct file. Then upload it with the UploadFileToStorageBucket tool: In_File = the attached paystub, in_emailAddress = the email address, in_FileName = <sanitized email>_paystub_{timestamp}.pdf for a PDF, or <sanitized email>_paystub_{timestamp}.png for a PNG. The extension must match the attached file. Keep the returned out_FileName.
   Rules for both uploads:
   - Build the sanitized email by replacing every "@" and every "." in the email address with "_". Example: jane.doe@example.com becomes jane_doe_example_com.
   - Write the text {timestamp} literally, including the curly braces. The tool replaces it with the real timestamp.
   - Ask for one file at a time. If the user attaches more than one file in a single message, do not guess which is which: ask them to send the files one at a time, starting with the application.
   - If an upload fails, tell the user and offer to try again; do not continue to step 4 until both uploads have succeeded.
4. Show a short summary: the email address and the two file names returned by the tool (application and paystub). Ask the user to confirm that the details are correct and that they want to submit the application.
5. Only after the user explicitly confirms, call the Start Benefit Claim tool once with in_Application = the out_FileName returned for the application, in_ExamplePaystub = the out_FileName returned for the paystub, and in_emailAddress = the email address. If the user does not confirm, ask what they want to change.
6. Tell the user that the application has been submitted and that they will be contacted by email about the next steps. Do not wait for the processing result, and do not guess or predict the outcome.

Tools:
- Eligibility context: use only for Job A questions, at most 3 searches per question.
- UploadFileToStorageBucket: call only in Job B steps 2 and 3, once per document, and only with a file the user attached in this conversation.
- Start Benefit Claim: call only in Job B step 5, after explicit user confirmation, and only once per application.
- Never invent, edit or reformat file names. Only pass out_FileName values exactly as returned by UploadFileToStorageBucket.

Privacy:
- Ask only for the information needed: the applicant's email and the two documents. Do not ask for social security numbers, bank details or other personal data.
- Do not repeat the contents of the uploaded documents back to the user.

Response style:
- Default to 2 to 4 sentences. Use short bullet lists for multi-part rules or summaries.
- Be warm and encouraging, but factual.

Uncertainty:
- If the Eligibility context does not cover something, say that you do not have that information and suggest contacting the benefits office.
- If a tool returns an error or an unexpected result, explain it simply and offer to retry. Never pretend that an action succeeded.
```

![Agent settings panel with the anthropic.claude-sonnet-5 model and the System prompt field filled in and highlighted](10-benefit-claims-assistant-conversational-agent.images/15-system-prompt.png){ .screenshot }

### 6. Test the agent: ask a question

Conversational agents are tested in the chat in Studio Web. Tool calls during the test are **real**: files really land in the bucket and the BPMN really starts.

Open the agent and start its test chat (**Debug**).

![Studio Web toolbar with the Debug button highlighted, and the Benefits Assistant Conversational Agent definition open](10-benefit-claims-assistant-conversational-agent.images/16-start-debug-W.png){ .screenshot width="900" }

Then type:

```text
What's the gross monthly income limit for a household of 4?
```

In the details of the run you see a search in the **Eligibility** context and no tool calls.

![Debug chat showing the Eligibility search, the agent's answer about the gross monthly income limit and its source in the Eligibility Criteria document](10-benefit-claims-assistant-conversational-agent.images/17-question-answer-W.png){ .screenshot width="900" }

Try a question the documents can't answer, for example "What's the weather tomorrow?", or a specific one like "Am I eligible? I earn $3,000." The agent should say it can't help with the first one. For the second, it should explain the general rules and invite you to submit an application, without saying whether you qualify.

### 7. Test the agent: register an application

1. Type:

    ```text
    I'd like to apply for SNAP benefits.
    ```

2. When the agent asks, give **your own** email address. You'll receive the final decision email there.
3. The agent asks for the **application**. Use the attach (paperclip) button in the chat and send [Sample application.pdf](dependencies/Sample%20application.pdf){ download="Sample application.pdf" }.
4. The agent uploads it and tells you the application was received. Then it asks for the **paystub**.
5. Attach your paystub (for example [Sample pay stub.png](dependencies/Sample%20pay%20stub.png){ download="Sample pay stub.png" }).
6. Read the summary. It should show your email and two file names.
7. Answer `yes`.

<div class="img-cols img-cols-50" style="padding: 0; gap: 1rem;" markdown>
  <div markdown>
![Debug chat for a new application: the email is confirmed, the application PDF is attached and uploaded with UploadFileToStorageBucket, and the paystub PNG is attached](10-benefit-claims-assistant-conversational-agent.images/18-registration-chat-1-W.png){ .screenshot style="width: 100%;" }
  </div>
  <div markdown>
![Debug chat with the paystub uploaded, the summary of the two stored file names, the yes answer, the Start Benefit Claim call and the submission confirmation](10-benefit-claims-assistant-conversational-agent.images/19-registration-chat-2-W.png){ .screenshot style="width: 100%;" }
  </div>
</div>

In the chat you should see:

- The agent calls **UploadFileToStorageBucket** twice, in two separate turns: first for the application, then for the paystub.
- The summary shows two **different** names, with a real timestamp, and the paystub keeps its own extension, for example:
    - `jane_doe_example_com_application_20261016143005.pdf`
    - `jane_doe_example_com_paystub_20261016143112.png`
- After your confirmation, the agent calls **Start Benefit Claim** once.
- The agent tells you the application was submitted and that you'll be contacted by email. It does **not** wait for a decision.

In **Orchestrator** (in the folder your debug run uses; ask your trainer if you're not sure which one):

| Where | What you should see |
|---|---|
| **Storage Buckets** → **Input Documents** | The two new files, with the names from the summary |
| **Jobs** | Two `UploadFileToStorageBucket` jobs and one `Start Benefit Claim` job, all **Successful** within seconds |
| **Jobs** | One `BenefitClaimsProcessing` job, **Running** |
| **Maestro** → process instances | The new `BenefitClaimsProcessing` instance with `in_Application`, `in_ExamplePaystub` and `in_emailAddress` filled with your values |

Later, after the case worker review, the approval or rejection email arrives in your inbox.

**Time to move to the next one!**
