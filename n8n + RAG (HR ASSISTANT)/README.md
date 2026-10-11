# PeopleDesk — HR Policy Knowledge Assistant

A Retrieval-Augmented Generation (RAG) chatbot built with **n8n** and **Ollama**. PeopleDesk is designed to answer employee questions about HR policies by retrieving relevant information from a collection of HR documents and using that content to generate a response.

> **Project status:** This README describes the intended workflow and the configuration discussed during development. Document indexing, model connectivity, and end-to-end behavior must be tested in the local n8n instance before the project is described as fully operational.

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Objectives](#2-objectives)
- [3. Technology Stack](#3-technology-stack)
- [4. How the System Works](#4-how-the-system-works)
- [5. Workflow Components](#5-workflow-components)
- [6. Knowledge-Base Preparation](#6-knowledge-base-preparation)
- [7. Configuration](#7-configuration)
- [8. Running the Project Locally](#8-running-the-project-locally)
- [9. Example Questions](#9-example-questions)
- [10. Testing and Validation](#10-testing-and-validation)
- [11. Limitations](#11-limitations)
- [12. Privacy and Responsible Use](#12-privacy-and-responsible-use)
- [13. Future Improvements](#13-future-improvements)
- [14. Repository Structure](#14-repository-structure)

## 1. Project Overview

PeopleDesk is an HR-focused adaptation of a document-based chatbot workflow originally designed for cybersecurity knowledge. The goal is to help employees find information in HR documents without manually searching through each file.

The assistant is intended to cover topics such as:

- Leave policies and application procedures
- Attendance and working hours
- Employee onboarding and joining formalities
- Employee benefits and eligibility conditions
- Workplace guidelines and company procedures

PeopleDesk uses a RAG approach. Instead of relying only on the language model's general knowledge, the workflow retrieves relevant passages from the configured HR knowledge base and passes them to the AI Agent as context for its answer.

## 2. Objectives

- Convert a cybersecurity knowledge assistant into an HR policy assistant.
- Use HR documents as the main source for policy-specific answers.
- Retrieve relevant document passages for employee questions.
- Support contextual follow-up questions through conversation memory.
- State clearly when the available documents do not contain enough information.
- Run the language model and embedding model locally through Ollama.

## 3. Technology Stack

| Technology / Component | Purpose |
|---|---|
| **n8n 2.38.1** | Visual workflow automation and orchestration. |
| **Ollama** | Runs local language and embedding models. |
| **Llama 3.2 (`llama3.2:latest`)** | Intended local chat model for generating responses. Verify that it is selected and connects successfully in n8n. |
| **Nomic Embed Text (`nomic-embed-text:latest`)** | Converts document chunks and questions into embeddings for semantic retrieval. |
| **Simple Vector Store** | Stores embedded document chunks and retrieves relevant content. |
| **Default Data Loader** | Loads document content and metadata for the ingestion process. |
| **Simple Memory** | Keeps conversational context for follow-up questions. |
| **n8n Chat Trigger** | Receives questions from the chat interface. |

The exact behavior depends on how the nodes and connections are configured in the local n8n workflow.

## 4. How the System Works

PeopleDesk has two logical processes: **document ingestion** and **question answering**. These processes work together but serve different purposes.

### 4.1 Document ingestion (prepare the knowledge base)

Before the assistant can answer from HR policies, the documents must be loaded and indexed.

1. **Prepare HR documents.** Gather approved HR policy documents, such as leave, attendance, onboarding, and benefits guides.
2. **Load the documents.** The Default Data Loader reads supported files and extracts document content and available metadata.
3. **Create embeddings.** The Ollama Embeddings node uses `nomic-embed-text:latest` to represent document content as numerical vectors. These vectors capture semantic information useful for finding related passages.
4. **Insert documents into the vector store.** The document content, embeddings, and available metadata are inserted into the Simple Vector Store using its document-insertion operation.
5. **Use a consistent store key.** The ingestion and retrieval configurations must use the same memory key so the assistant searches the intended HR knowledge base.

The intended HR knowledge-base key is:

```text
hr_policy_knowledge
```

**Important:** Renaming the retrieval description or memory key does not automatically replace old cybersecurity content. The HR files must actually be indexed under the HR key, and retrieval must be tested to confirm it returns HR content.

### 4.2 Question answering (retrieve and respond)

When an employee submits a question, the intended sequence is:

1. **Chat Trigger receives the question.** The trigger starts the workflow with the user's message.
2. **AI Agent interprets the request.** The HR Policy Assistant identifies the topic and decides when the vector-store retrieval tool is relevant.
3. **Vector Store retrieves relevant passages.** The agent calls the Simple Vector Store in `Retrieve Documents (As Tool for AI Agent)` mode. The store searches the configured HR knowledge base and returns relevant document chunks, up to the configured result limit.
4. **The chat model prepares an answer.** The Ollama chat model receives the question and retrieved context and generates a readable response.
5. **Memory supports follow-up questions.** Simple Memory helps preserve conversational context, for example when a user asks, “Which of those documents must be submitted before joining?”
6. **The response is shown in chat.** The user receives the answer through the n8n chat interface.

### 4.3 Architecture overview

```text
DOCUMENT INGESTION
==================
HR Policy Files
      |
      v
Default Data Loader
      |
      v
Ollama Embeddings
(nomic-embed-text:latest)
      |
      v
Simple Vector Store
Insert Documents
Key: hr_policy_knowledge


QUESTION ANSWERING
==================
Employee Question
      |
      v
When Chat Message Received
      |
      v
HR Policy Assistant (AI Agent)
      |
      +------------------------------+
      |                              |
      v                              v
Ollama Chat Model              Simple Memory
(llama3.2:latest)               (follow-up context)
      |
      | AI Agent also calls the retrieval tool
      v
Simple Vector Store
Retrieve Documents
Key: hr_policy_knowledge
      |
      v
Relevant HR Passages
      |
      v
Grounded Answer in Chat
```

The diagram describes the intended logical flow. The actual n8n canvas must be checked to confirm that the ingestion nodes are connected and that indexing completes successfully.

## 5. Workflow Components

### 5.1 When Chat Message Received

This is the entry point for employee questions. The AI Agent's prompt input is configured to use the incoming chat message:

```javascript
{{ $json.chatInput }}
```

The chat trigger can also provide the chat interface's title, subtitle, greeting, and input placeholder.

Suggested interface text:

- **Title:** PeopleDesk — HR Policy Assistant
- **Subtitle:** Ask questions about HR policies and employee procedures.
- **Greeting:** Hi! I'm PeopleDesk, your HR Policy Assistant. Ask me about leave, attendance, onboarding, benefits, or workplace policies.
- **Input placeholder:** Ask an HR policy question...

### 5.2 HR Policy Assistant (AI Agent)

The AI Agent coordinates question interpretation, retrieval, and answer generation. Its system message instructs it to:

- Answer questions about HR policies and procedures.
- Use the vector-store retrieval tool for policy-specific questions.
- Base policy claims on retrieved content.
- Mention a source document when source metadata is available.
- Admit when the knowledge base does not support a definitive answer.
- Explain conflicting or ambiguous policy information rather than inventing a rule.
- Maintain context for follow-up questions.
- Protect employee privacy and avoid making HR decisions.

The agent is an informational assistant; it does not approve leave, make employment decisions, or guarantee benefits.

### 5.3 Simple Vector Store

The vector store has two distinct roles, configured through its operation mode:

- **Insert Documents:** used in the ingestion process to add HR document content and embeddings.
- **Retrieve Documents (As Tool for AI Agent):** used during question answering so the agent can search the indexed content.

Planned retrieval settings:

| Setting | Intended value |
|---|---|
| Operation Mode | Retrieve Documents (As Tool for AI Agent) |
| Memory Key | `hr_policy_knowledge` |
| Limit | `4` |
| Include Metadata | Enabled |
| Rerank Results | Disabled for the initial prototype |

The tool description should explain that it searches HR policy documents and should not be used to invent answers.

### 5.4 Default Data Loader

The Default Data Loader supplies document text and metadata to the ingestion path. Configure it to read the intended HR files and verify that it extracts readable content. The exact source configuration depends on the file format and the nodes used in the workflow.

### 5.5 Ollama Embeddings

The embedding node is intended to use:

```text
nomic-embed-text:latest
```

The same embedding model should be used consistently for indexing and retrieval. If the embedding model changes, previously indexed vectors may need to be recreated to ensure compatible retrieval.

### 5.6 Ollama Chat Model

The intended chat model is:

```text
llama3.2:latest
```

Confirm that the model is selected in n8n and that the Ollama credential test succeeds. The chat model generates the final natural-language response; the embedding model serves a different purpose and does not generate the answer itself.

### 5.7 Simple Memory

Simple Memory supports conversational context between messages in a chat session. Configure the memory session key using the appropriate session identifier from the Chat Trigger. Follow-up questions should be tested within the same session.

Memory is not the HR knowledge base. Policy-specific facts should still be verified against retrieved documents.

### 5.8 Read/Write Files from Disk

The workflow may include a Read/Write Files from Disk node to access local files. Its presence on the canvas alone does not mean the documents have been indexed. Ensure that the file-reading path is correctly connected to the document ingestion process.

## 6. Knowledge-Base Preparation

For a demonstration, the HR knowledge base can include the following sample documents:

| Suggested filename | Example content |
|---|---|
| `01_Leave_Policy.pdf` | Leave categories, eligibility, application steps, notice requirements, and approvals. |
| `02_Attendance_Policy.pdf` | Working hours, attendance recording, late arrivals, and regularization. |
| `03_Onboarding_Guide.pdf` | Joining formalities, required documents, induction, and employee setup. |
| `04_Benefits_Policy.pdf` | Benefits, eligibility criteria, claims, and applicable conditions. |

These filenames and topics are suggestions, not documents supplied with this repository. Add the actual files you are authorized to use.

For academic demonstrations, clearly labelled fictional policies are suitable when real company policies are unavailable. Do not upload confidential employee records or personal information.

## 7. Configuration

### 7.1 Prerequisites

- Windows computer with n8n installed.
- Ollama installed and running.
- The intended chat and embedding models downloaded.
- HR policy documents prepared.
- n8n workflow nodes and credentials configured.

The local Ollama model list previously showed these models:

```text
llama3.2:latest
nomic-embed-text:latest
```

Check current availability before running the workflow.

### 7.2 Verify Ollama

In PowerShell, run:

```powershell
ollama list
```

Check that the models are available. You can also test the local Ollama API:

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/tags"
```

For a direct Windows setup where n8n and Ollama both run on the host, try this base URL in the n8n Ollama credential:

```text
http://127.0.0.1:11434
```

A previous attempt returned `ECONNREFUSED ::1:11434`, indicating a refused connection to the IPv6 localhost address. The IPv4 URL above is a useful test for a direct Windows setup. If n8n runs inside Docker or another environment, its correct Ollama host address may be different.

### 7.3 Start n8n

Start n8n using the method used for your local installation. If installed globally through npm, a common command is:

```powershell
n8n
```

Then open the local n8n interface, commonly:

```text
http://localhost:5678
```

Use the actual URL and port configured in your environment if they differ.

### 7.4 Configure the nodes

1. Open the PeopleDesk workflow.
2. Configure the Chat Trigger's title, subtitle, greeting, and input placeholder.
3. Set the AI Agent system message to the HR-specific instructions.
4. Connect the Ollama Chat Model to the AI Agent's Chat Model input.
5. Configure Ollama Embeddings with `nomic-embed-text:latest`.
6. Configure the Default Data Loader to read the HR documents.
7. Set up the ingestion operation to insert the loaded HR documents into the Simple Vector Store.
8. Use `hr_policy_knowledge` consistently for both document insertion and retrieval.
9. Configure the retrieval tool description and set its result limit to 4 for the initial prototype.
10. Configure Simple Memory to use the chat session identifier.
11. Execute the ingestion process and confirm that documents are inserted.
12. Test the chat workflow and inspect the execution output if any node fails.

**Credential safety:** Do not commit API keys, passwords, authentication tokens, or private credentials to GitHub. The current design intends to use local Ollama models; if any external service is added later, keep its credentials in n8n's credential manager or an appropriate secret-management system.

## 8. Example Questions

Use questions whose answers are actually present in the indexed documents.

### Example 1 — Simple policy question

**Question**

> According to the uploaded leave policy, how do I apply for annual leave?

**Expected behavior**

PeopleDesk retrieves the relevant leave-policy passage and summarizes the documented process. It should not invent approval levels or deadlines.

### Example 2 — Detailed onboarding question

**Question**

> What steps must a new employee complete during onboarding, and which documents are required according to the uploaded guide?

**Expected behavior**

PeopleDesk organizes the steps and documents found in the guide and preserves any stated conditions or exceptions.

### Example 3 — Follow-up question

**First question**

> According to the uploaded onboarding guide, what documents are required?

**Follow-up**

> Which of those must be submitted before the joining date?

**Expected behavior**

Simple Memory helps the assistant understand what “those” refers to. The assistant must still retrieve or verify the timing requirement in the source document. If the guide does not specify it, PeopleDesk should say so.

### Example 4 — Information not present in the knowledge base

**Question**

> Does the uploaded HR documentation specify a relocation allowance?

**Expected behavior**

If the documents do not cover relocation allowance, PeopleDesk should clearly say that the answer cannot be confirmed from the available HR documentation.

These are test scenarios, not recorded outputs from a completed run. Capture actual results after executing the workflow.

## 9. Testing and Validation

Before presenting the project as complete, validate the following:

- [ ] The Ollama credential connects successfully from n8n.
- [ ] The selected chat model generates a response.
- [ ] The embedding model works.
- [ ] HR documents are loaded and inserted into the vector store.
- [ ] Ingestion and retrieval use the same key: `hr_policy_knowledge`.
- [ ] Retrieval returns HR content rather than leftover cybersecurity content.
- [ ] The chatbot answers a simple policy question using retrieved evidence.
- [ ] A detailed question returns relevant policy details without unsupported additions.
- [ ] A follow-up question works in the same chat session.
- [ ] The assistant acknowledges when information is missing.
- [ ] Source metadata is included when available.
- [ ] The workflow finishes without node or credential errors.

For a strong project demonstration, capture screenshots of document insertion, the full workflow canvas, the chatbot interface, a first query and response, a detailed query, and a memory follow-up.

## 10. Limitations

- **In-memory storage:** The Simple Vector Store is intended for prototyping. Its n8n node warning states that data is stored in memory and may be lost when n8n restarts or memory is cleared.
- **Document dependency:** Answer quality depends on the accuracy, completeness, and freshness of the HR documents.
- **Retrieval quality:** Chunking, embeddings, metadata, and retrieval limits can affect which passages are found.
- **Model connectivity:** The Ollama endpoint and model selection must be verified in the actual environment.
- **No automatic policy authority:** The assistant does not determine which company policy is legally or operationally authoritative when documents conflict.
- **No live HR system integration:** The described workflow does not itself connect to payroll, employee records, leave-management systems, or an HRIS.
- **Implementation verification:** A configured canvas is not proof of successful end-to-end operation; ingestion and chat tests are required.

## 11. Privacy and Responsible Use

- Use approved or fictional sample HR documents.
- Do not upload sensitive employee information without authorization.
- Do not expose credentials or tokens in prompts, screenshots, or repository files.
- Do not present unsupported generated content as official policy.
- Refer ambiguous or conflicting policy questions to HR.
- Treat the chatbot as an informational prototype, not an automated decision-maker.

## 12. Future Improvements

Possible improvements after the initial prototype include:

1. Replace in-memory storage with a persistent vector database.
2. Display source document names and page or section details where available.
3. Add a repeatable process for indexing updated policy documents.
4. Test retrieval against a fixed set of questions and expected evidence.
5. Add role-based access controls before introducing non-public HR content.
6. Add logging and error handling without storing unnecessary sensitive information.
7. Evaluate different chunk sizes and retrieval limits to improve answer relevance.

## 13. Repository Structure

A suggested repository layout is:

```text
PeopleDesk-HR-RAG/
├── README.md
├── workflow/
│   └── peopledesk_hr_rag.json
├── documents/
│   └── README.md
└── screenshots/
    ├── workflow-architecture.png
    ├── document-insertion.png
    ├── chatbot-interface.png
    ├── first-query-response.png
    ├── detailed-query.png
    └── memory-follow-up.png
```

This is a suggested layout. Add the workflow export and screenshots only after creating them. Include HR documents only if you have permission to redistribute them; otherwise, document how users can provide their own local files.

## 14. Conclusion

PeopleDesk adapts a document-grounded chatbot architecture to the HR domain. n8n coordinates the workflow, Ollama provides local language and embedding models, the Default Data Loader prepares HR documents, and the Simple Vector Store supports semantic retrieval. The AI Agent uses retrieved policy content to produce answers, while Simple Memory supports contextual follow-ups.

The central design principle is that **HR policy answers should be grounded in the indexed documents**. If the evidence is absent or unclear, the assistant should say so instead of inventing a company rule.

---

**Project:** PeopleDesk — HR Policy Knowledge Assistant  
**Workflow platform:** n8n  
**Local model runtime:** Ollama  
**Approach:** Retrieval-Augmented Generation (RAG)
