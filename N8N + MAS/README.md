# FinWise – Finance Multi-Agent System

## 1. Project Overview

FinWise is an AI-powered finance assistant built using **n8n, Ollama, and Tavily Search**. It uses a sequential multi-agent architecture to process financial queries, research relevant information, analyze financial implications and risks, and generate a structured final response.

The system is designed to provide financial education, research support, and risk-aware insights through a conversational interface.

## 2. Objectives

- Build a sequential multi-agent workflow using n8n.
- Divide financial problem-solving into specialized agent roles.
- Retrieve relevant external financial information when required.
- Analyze financial implications, uncertainties, and risks.
- Generate clear and structured responses to users.
- Explore local LLM integration using Ollama.

## 3. System Architecture

The workflow consists of three specialized AI agents connected sequentially.

| Stage | Input | Output |
|---|---|---|
| Financial Research | User's chat message | Research brief |
| Financial Analysis | Research brief | Risk and implication analysis |
| Financial Recommendations | Risk analysis | Final user-facing answer |

**Workflow:**

`Chat Trigger → Financial Research → Financial Analysis → Financial Recommendations`

The Ollama Chat Model is shared across the three agents. Tavily Search is connected as an optional research tool, while Simple Memory supports conversational context.

## 4. Technology Stack

- **n8n:** Workflow automation and agent orchestration.
- **Ollama:** Local LLM inference.
- **Llama 3.2:** Intended chat model (`llama3.2:latest`).
- **Tavily Search:** External web research.
- **Simple Memory:** Conversational context retention.

## 5. Agent Responsibilities

### Agent 1: Financial Research

The Financial Research Agent analyzes the user's question and identifies the relevant financial domain. It researches facts, financial concepts, market developments, company information, or regulatory updates when required.

It produces a structured research brief for the next agent.

### Agent 2: Financial Analysis

The Financial Analysis Agent evaluates the research findings and identifies relevant financial implications, assumptions, uncertainties, and risks.

Depending on the query, it considers market volatility, liquidity, inflation, interest rates, investment risks, and potential downside scenarios.

It produces a structured financial risk and implication analysis.

### Agent 3: Financial Recommendations

The Financial Recommendations Agent synthesizes the preceding analysis into a final user-facing response.

The response may include key findings, financial implications, important risks, practical options, assumptions, limitations, and available source references.

## 6. Setup and Configuration

### Prerequisites

- A local n8n installation.
- Ollama installed and running.
- The `llama3.2:latest` model available in Ollama.
- A Tavily API key.
- Appropriate n8n credentials for Ollama and Tavily.

### Step 1: Start Ollama

Verify that Ollama is running and list the installed models:

```bash
ollama list
```

If required, start a model using:

```bash
ollama run llama3.2:latest
```

### Step 2: Start n8n

Run the following command in a terminal:

```bash
n8n
```

Open the local n8n interface:

`http://localhost:5678`

### Step 3: Import the Workflow

1. Open n8n.
2. Select the option to import a workflow from a file.
3. Import the FinWise workflow JSON.
4. Review the nodes and their connections.

### Step 4: Configure Ollama

Configure the Ollama Chat Model node with the installed model:

`llama3.2:latest`

For a direct Windows installation of both n8n and Ollama, the usual base URL is:

`http://127.0.0.1:11434`

If n8n runs in a container or a different environment, configure the appropriate Ollama endpoint.

### Step 5: Configure Tavily

Create or select the Tavily credential in n8n and provide the API key through the credential settings. Do not hard-code API keys into workflow prompts or commit them to version control.

### Step 6: Configure Simple Memory

Configure Simple Memory to use the chat session identifier and an appropriate context window. Verify that follow-up messages use the same session identifier when conversational continuity is required.

### Step 7: Test the Workflow

Save the workflow and test it through the n8n chat interface. Confirm that the agents execute in sequence and the final response is returned successfully.

## 7. Example Test Query

The following query can be used to test the workflow:

> Explain SIP investing and estimate the future value of a ₹5,000 monthly SIP over 10 years, assuming a 12% annual return.

Expected behavior:

- The Financial Research Agent identifies relevant SIP concepts.
- The Financial Analysis Agent explains the assumptions, projection methodology, and investment risks.
- The Financial Recommendations Agent presents the findings in a clear final response.

Any projected investment value must be labelled as hypothetical and must not be presented as a guaranteed return.

## 8. Key Features

- Sequential multi-agent processing.
- Specialized prompts for research, analysis, and recommendations.
- Local LLM integration through Ollama.
- Optional external research using Tavily Search.
- Conversational memory support.
- Structured financial responses.
- Explicit consideration of uncertainty and financial risk.

## 9. Limitations

- Output quality depends on the selected language model and its capabilities.
- External research depends on search availability and source quality.
- Tavily Search does not itself guarantee accurate live market prices or financial data.
- Financial projections depend on the assumptions used.
- Conversational memory depends on correct session configuration.
- The system is intended for educational and research purposes, not as a substitute for qualified financial advice.

## 10. Future Enhancements

- Integrate dedicated market-data APIs for current stock prices and financial indicators.
- Add Python-based financial calculators for deterministic numerical results.
- Implement retrieval-augmented generation (RAG) using reliable financial documents.
- Add a dedicated fact-checking and source-validation stage.
- Improve session management and persistent memory.
- Add automated testing, logging, and evaluation of agent outputs.

## 11. Project Status

The workflow structure and finance-specific agent prompts have been prepared. The Ollama credential connection, Tavily configuration, memory behavior, and end-to-end execution must be verified in the local n8n environment before the project can be considered fully operational.

## 12. Disclaimer

FinWise provides financial education and research support. It does not guarantee investment performance or replace professional financial advice. Users should independently verify important financial information before making financial decisions.
