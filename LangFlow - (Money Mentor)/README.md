# 💰 Money Mentor — AI-Powered Personal Budgeting Assistant

Money Mentor is a conversational financial-literacy assistant built in **Langflow**. It uses a **Gemini language model through the Gemini API** and a **Calculator tool** to help users understand their monthly income, create a budget, and perform basic financial calculations.

The project demonstrates how a visual LLM workflow can combine natural-language understanding with a calculation tool to generate structured, easy-to-understand budgeting guidance.

> **Example:** A user says, “I earn ₹40,000 a month. Make me a 50/30/20 budget.” Money Mentor calculates the suggested allocations and explains how to use them.

---

## ✨ Features

- **Conversational interface:** Users can describe their income and budgeting needs in natural language.
- **Gemini-powered responses:** The LLM interprets the request and generates a clear, human-friendly explanation.
- **Calculator tool integration:** The agent can use a calculator for arithmetic instead of relying only on mental calculation by the language model.
- **50/30/20 budget breakdown:** Helps divide take-home income into needs, wants, and savings/investments.
- **Practical guidance:** Provides examples of expenses and simple steps to get started.
- **Visual workflow:** The agent is assembled and connected in Langflow, making the flow easier to inspect and modify.
- **Optional web context:** The illustrated workflow also includes Web Search and URL components connected to the Agent. These can provide external search results and extracted webpage content when configured and used by the agent.

---

## 🧰 Technology Stack

| Component | Purpose |
|---|---|
| Langflow | Visual builder for connecting the workflow components |
| Gemini API / Gemini model | Understands user requests and generates conversational responses |
| Agent component | Coordinates the LLM, instructions, user input, and connected tools |
| Calculator tool | Performs arithmetic required for budget calculations |
| Chat Input | Receives the user's message from the Langflow Playground |
| Chat Output | Displays the final response |
| Web Search component *(if configured and invoked)* | Retrieves relevant web search results |
| URL component *(if configured and invoked)* | Fetches content from one or more web pages, subject to its configuration |

*The exact component settings, model version, API provider configuration, and tool behavior depend on the Langflow project configuration.*

---

## 🏗️ Architecture and Workflow

The workflow shown in Langflow contains a Chat Input, an Agent, connected tools, and a Chat Output. The Agent is configured to use a Gemini model. A Calculator tool is part of the toolset described for this project.

### High-level flow

```text
                  ┌─────────────────────┐
                  │     User Message    │
                  │     (Chat Input)    │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │        Agent        │
                  │ Gemini LLM +        │
                  │ Agent Instructions  │
                  └──────────┬──────────┘
                             │
                 ┌───────────┼────────────┐
                 │           │            │
                 ▼           ▼            ▼
          ┌────────────┐ ┌──────────┐ ┌─────────────┐
          │ Calculator │ │ Web      │ │ URL Content │
          │ Tool       │ │ Search   │ │ Extraction  │
          └─────┬──────┘ └────┬─────┘ └──────┬──────┘
                └─────────────┼──────────────┘
                              │
                              ▼
                  ┌─────────────────────┐
                  │ Final Agent Response│
                  │    (Chat Output)    │
                  └─────────────────────┘
```

**Important:** The Calculator is described as part of the configured toolset. The screenshot visibly shows Web Search and URL components connected to the Agent; verify that the Calculator is also connected and enabled in your saved Langflow project. Web Search and URL retrieval are only used when the Agent decides to invoke them and their components are correctly configured.

### Step-by-step execution

1. **User submits a message**  
   The Chat Input component receives the request from the Langflow Playground. For example: `I earn ₹40,000 a month. Make me a 50/30/20 budget.`

2. **The Agent receives the request**  
   The Agent combines the user's message with its configured instructions and passes the task to the selected Gemini model.

3. **The LLM interprets the task**  
   The model identifies the relevant information, such as the monthly income, the requested budgeting rule, and the expected format of the answer.

4. **The Agent can call tools**  
   When arithmetic or external information is needed, the Agent may invoke an available tool. The Calculator performs arithmetic; Web Search and URL retrieval can provide additional context if they are configured and selected for the task.

5. **Tool results return to the Agent**  
   The Agent uses the returned results to formulate a response. Tool invocation depends on the Agent's instructions, tool descriptions, and runtime behavior.

6. **The Agent generates the final answer**  
   Gemini turns the relevant calculations and context into an understandable budget breakdown, examples, and practical tips.

7. **Chat Output displays the response**  
   The final response is returned to the Playground for the user to read.

---

## 🧮 Example: 50/30/20 Budget

The 50/30/20 rule is a simple budgeting guideline that allocates take-home income across three categories:

- **50% — Needs:** Essential expenses such as rent, groceries, utilities, transport, and basic bills.
- **30% — Wants:** Discretionary expenses such as dining out, shopping, entertainment, and subscriptions.
- **20% — Savings and investments:** Savings goals, an emergency fund, recurring deposits, or suitable investments.

For a monthly take-home income of **₹40,000**, the allocations are:

| Category | Allocation | Calculation | Monthly amount |
|---|---:|---:|---:|
| Needs | 50% | ₹40,000 × 0.50 | ₹20,000 |
| Wants | 30% | ₹40,000 × 0.30 | ₹12,000 |
| Savings and investments | 20% | ₹40,000 × 0.20 | ₹8,000 |
| **Total** | **100%** | | **₹40,000** |

### How the Calculator helps

The Agent can use the Calculator tool to evaluate the arithmetic required for the allocations. For example:

```text
Needs   = 40000 * 0.50 = 20000
Wants   = 40000 * 0.30 = 12000
Savings = 40000 * 0.20 =  8000
```

This separates arithmetic from the LLM's natural-language explanation. The Calculator returns numerical results, while the LLM explains what those results mean. The exact tool-call sequence depends on the saved Agent configuration.

### Example response

> Here is a simple 50/30/20 budget tailored to your monthly take-home income of ₹40,000.
>
> - **Needs — ₹20,000:** Rent, groceries, electricity, transport, and essential bills.
> - **Wants — ₹12,000:** Dining out, shopping, entertainment, and subscriptions.
> - **Savings and investments — ₹8,000:** Build an emergency fund and work toward savings goals before choosing investments.
>
> Try setting aside the savings amount when your salary arrives. Adjust the percentages if your essential expenses are unusually high.

The response shown in the demonstration also suggests building an emergency fund covering roughly 3–6 months of essential expenses. Treat such suggestions as general budgeting guidance and tailor them to individual circumstances.

---

## ⚙️ Setup and Configuration

These steps describe the general setup. Component names and menus can vary by Langflow version.

### Prerequisites

- A Langflow instance (local or hosted)
- A Google AI Studio / Gemini API key with access to the selected model
- A Calculator component/tool available in Langflow
- Optional: correctly configured Web Search and URL components if you want the agent to retrieve external information

### 1. Open the Langflow project

Open your Langflow instance and create a new flow or import your existing flow. Add or verify these components:

- Chat Input
- Agent
- Gemini language model configuration
- Calculator tool
- Chat Output
- Optional Web Search and URL components

### 2. Configure the Gemini API

1. Obtain an API key from Google AI Studio.
2. Select the Gemini model supported by your Langflow integration.
3. Add the API key through the supported Langflow credential or environment-variable mechanism.
4. Confirm that the Agent's Language Model field points to the intended Gemini model.
5. Run a simple test to confirm that the model can respond.

**Security:** Never commit API keys to GitHub, paste them into screenshots, or hard-code them into public flow exports. Use Langflow's supported secret-management mechanism or environment variables.

If your installation supports environment-based configuration, a local `.env` file may look like this:

```env
GOOGLE_API_KEY=your_gemini_api_key_here
```

This is an illustrative variable name; use the exact variable or credential field expected by your Langflow component. Add `.env` to `.gitignore`.

### 3. Configure the Agent instructions

Set the Agent instructions to define its role, calculation expectations, communication style, and limitations. For example:

```text
You are Money Mentor, a friendly personal budgeting assistant.

Help users understand their income, expenses, savings goals, and
basic budgeting concepts in clear, practical language.

For budgeting requests:
1. Identify the income amount and the budgeting rule requested.
2. Use the Calculator tool for arithmetic when available.
3. Clearly show the percentages, calculations, and resulting amounts.
4. Check that the category amounts add up to the total income.
5. Explain categories with realistic examples.
6. Ask a clarifying question if important information is missing.
7. State assumptions, such as whether the income is take-home pay.
8. Do not guarantee investment returns or present general guidance
   as individualized regulated financial advice.
9. Do not invent facts, search results, or tool outputs.
```

Adapt these instructions to the exact tools and inputs in your flow.

### 4. Configure and connect the Calculator

1. Add the Calculator component/tool available in your Langflow installation.
2. Configure any required inputs or settings.
3. Connect it to the Agent's **Tools** input, not to the final Chat Output.
4. Check that the tool is enabled and has a clear description so the Agent knows when to use it.
5. Test it with simple arithmetic before testing a full budgeting request.

The Calculator's exact implementation and input format depend on the component version. Avoid assuming that a tool is active merely because it exists somewhere on the canvas.

### 5. Configure optional web components

If you intend to use Web Search and URL extraction:

- Connect each component to the Agent's Tools input.
- Configure required credentials, endpoints, and allowed actions.
- Test search and URL extraction independently.
- Instruct the Agent to use external sources only when relevant, and to distinguish sourced information from estimates.

The URL component shown in the workflow describes fetching content from one or more web pages and following links recursively according to its depth setting. Use sensible depth limits and only retrieve content you have permission to access.

### 6. Connect the workflow

Verify the data flow:

```text
Chat Input → Agent → Chat Output
Calculator ────────→ Agent Tools
Web Search ────────→ Agent Tools  (optional)
URL Component ────→ Agent Tools  (optional)
Gemini Model ─────→ Agent Language Model
```

The model connection and tool connections are different: the language model generates and interprets language, while tools provide specific capabilities.

### 7. Test in the Playground

Open **Playground** in Langflow and try the sample prompts below. Confirm that the flow executes successfully, tool calls return results when needed, and the final response is readable.

---

## 🧪 Suggested Test Cases

| Test prompt | What to verify |
|---|---|
| `I earn ₹40,000 a month. Make me a 50/30/20 budget.` | Correct ₹20,000 / ₹12,000 / ₹8,000 allocations |
| `I earn ₹60,000 monthly. Calculate 20% savings.` | Savings amount is ₹12,000 |
| `My income is ₹35,000. Use the 50/30/20 rule.` | Allocations total ₹35,000 |
| `Explain the difference between needs and wants.` | Clear explanation without unnecessary calculations |
| `I have irregular income. How should I budget?` | Sensible caveats and a request for relevant context |
| `Find current official guidance on a financial topic.` | Web Search is invoked only if configured and relevant; claims are grounded in retrieved information |

For calculation tests, manually verify the expected values rather than relying solely on the LLM's response.

---

## 🛡️ Limitations and Responsible Use

- **Educational purpose:** Money Mentor is a budgeting and financial-literacy demonstration, not a substitute for a qualified financial adviser.
- **No guaranteed outcomes:** It must not guarantee investment returns or imply that a budgeting rule fits every individual.
- **Context matters:** Rent, debt, dependants, emergency savings, and irregular income can make a strict 50/30/20 split impractical.
- **LLM reliability:** The model can misunderstand a request or produce incorrect explanations. Validate important calculations and advice.
- **Tool availability:** Calculator, search, and URL tools work only when properly configured and available at runtime.
- **External information:** Search results and webpage content may be incomplete, stale, or inaccurate. Verify important information with authoritative sources.
- **Privacy:** Avoid entering account numbers, passwords, payment details, or other sensitive personal information.
- **API usage:** Gemini API quotas, billing, model availability, and rate limits depend on the account and selected model.

---

## 🔒 Recommended `.gitignore`

If you export or maintain supporting files locally, exclude secrets and local configuration:

```gitignore
.env
.env.*
!.env.example
__pycache__/
*.py[cod]
.venv/
venv/
```

An `.env.example` may document the required variable name without containing a real secret:

```env
GOOGLE_API_KEY=
```

Only include flow exports after checking that they contain no credentials, private endpoints, personal data, or other secrets.

---

## 🚀 Future Enhancements

- Add an expense-entry workflow and compare actual spending against the planned budget.
- Allow users to customize category percentages and savings goals.
- Add recurring monthly summaries and budget-versus-actual analysis.
- Introduce validation checks to ensure all allocations sum to the user's income.
- Provide source citations when the Agent uses external financial information.
- Add unit tests for budget calculations and edge cases, including zero, negative, missing, and unusually large inputs.
- Add a clear disclaimer and a path to qualified professional advice for complex financial decisions.

---

## 📁 Suggested Repository Structure

```text
money-mentor/
├── README.md
├── flow/
│   └── money-mentor-flow.json   # Optional, sanitized Langflow export
├── screenshots/
│   ├── workflow.png             # Optional workflow screenshot
│   └── demo.png                 # Optional Playground screenshot
├── .env.example
└── .gitignore
```

This is a suggested structure, not a claim that these files are already present. Export the flow from Langflow and add screenshots only if you want to include them in the repository.

---

## 📌 Project Summary

Money Mentor demonstrates a practical **LLM + tools** pattern using Langflow. A user sends a natural-language budgeting request; the Agent interprets it using Gemini, can delegate arithmetic to a Calculator tool, and returns an understandable budget through Chat Output. Optional web tools can extend the workflow with external context when configured.

The key design principle is to use the LLM for understanding and explanation, use tools for tasks they are designed to perform, and validate the resulting numbers before relying on them.

---

## 📄 Disclaimer

This project is for educational and demonstration purposes only. It does not provide individualized investment, tax, legal, or other regulated financial advice. Users should verify calculations and consult an appropriately qualified professional when needed.
