Build Homework 3 for my AI Agents course: "Support Ticket Router: Your First LLM Inside a Graph".

Use Python, LangGraph, LangChain OpenAI, Pydantic, and a Google Colab notebook.

First create prompt.md containing this complete technical specification. Then create support_ticket_router.ipynb. Optionally create README.md with run instructions.

GOAL
TaskNest is a small to-do application. Build a customer support ticket router in which an OpenAI language model classifies each ticket into exactly one of three categories, and a LangGraph conditional edge routes the ticket to the correct team.

CATEGORIES
- billing: payments, invoices, refunds, subscriptions, charges.
- technical: bugs, crashes, error messages, login problems, app failures.
- general: feature requests, general questions, appreciation or thank-you messages.

IMPORTANT AMBIGUITY
A ticket such as "The app crashes every time I try to pay" describes a technical failure even though it mentions payment. The LLM should consider the whole meaning of the ticket, not merely the presence of keywords. Do not implement manual keyword classification or override the LLM's classification with if-statements.

REQUIRED GRAPH
Build exactly four functional nodes:
1. classify
2. billing_team
3. tech_team
4. general_team

Connect them as follows:
START -> classify
classify -> exactly ONE conditional routing edge with three branches:
  billing -> billing_team
  technical -> tech_team
  general -> general_team
Each team node -> END.

No other functional nodes, no loops, no memory, no external tools, no additional LLM calls.

STATE
Use a TypedDict containing exactly the four required fields:
- ticket: str, the original input text
- category: Literal["billing", "technical", "general"], chosen by the LLM
- team: str, set by the relevant team node
- reply: str, set by the relevant team node

THE LLM NODE
- classify is the only node that calls an LLM.
- Use ChatOpenAI from langchain_openai.
- Use a low-cost OpenAI model, with a single MODEL_NAME configuration constant. Default to gpt-6-luna, which I used in Homework 1 and which the lecturer lists as an example. Explain this choice in prompt.md; if the model is unavailable or incompatible, report the problem rather than silently choosing another model.
- Set temperature=0 for maximum consistency, where supported.
- Create a Pydantic BaseModel with category: Literal["billing", "technical", "general"].
- Use ChatOpenAI(...).with_structured_output(...) to ensure a valid category.
- In classify, read state["ticket"], send an instruction explaining all three category definitions and the need to infer the primary issue, make exactly ONE LLM invocation, and return only {"category": chosen_category}.
- Do not generate a free-form category and then manually parse it.

THE CONDITIONAL EDGE
- Write a routing function route(state) that does not call the model.
- It must only read state["category"] and return that category.
- Use exactly one add_conditional_edges call from classify mapping each category to the corresponding team node.

TEAM NODES
Each team node must be ordinary deterministic Python without any LLM call.
Each must return only team and reply.
Use fixed short customer-friendly messages:
- Billing Team: "Your ticket has been forwarded to our Billing Team. We will review your billing question."
- Technical Support Team: "Your ticket has been forwarded to our Technical Support Team. We will investigate the issue."
- General Support Team: "Your ticket has been forwarded to our General Support Team. Thank you for contacting us."

API KEY SECURITY
- In Google Colab, load the API key from the Secrets panel using:
  import os
  from google.colab import userdata
  os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_API_KEY")
- The secret must be named OPENAI_API_KEY and notebook access must be enabled.
- Never use getpass.
- Never hardcode or print API keys.
- Never commit secrets, .env files, or credentials to GitHub.
- Explain how to configure Colab Secrets in a markdown cell.
- If the secret is absent, provide a helpful non-sensitive error message.

NOTEBOOK REQUIREMENTS
- Include a setup cell: %pip install -q langgraph langchain-openai
- Explain the purpose of the State, classify node, Pydantic structured output, conditional routing function, and team nodes in beginner-friendly language.
- Compile the graph and name it graph.
- Draw the actual compiled graph using graph.get_graph().draw_mermaid_png() and display it. If the renderer is unavailable, show the Mermaid source without preventing the rest of the notebook from running.
- Support exactly this call:
  result = graph.invoke({"ticket": "I was charged twice this month"})
- Every result must include: ticket, category, team, reply.
- Print the category, team, and reply for each test.

TESTS
Add at least 9 distinct test tickets, including:
Billing:
1. "I was charged twice this month."
2. "Where can I download my invoice?"
3. "I want a refund for my subscription."
Technical:
4. "I cannot log into my account."
5. "The app crashes when I create a task."
6. "The app crashes every time I try to pay."
General:
7. "Could you add dark mode?"
8. "Thanks for your excellent support!"
9. "Does TaskNest have keyboard shortcuts?"

The ambiguous payment/crash example must be discussed explicitly.

For each test:
- Run graph.invoke exactly once (therefore one LLM call per ticket).
- Print the original ticket, category, team and reply.
- Assert that all four required keys exist.
- Assert category is one of the three supported values.
- Assert that the selected team and fixed reply correspond to that category.
- Show counts of observed classifications.
- Check whether at least two tickets were routed to each team.
- Do not falsify results or silently change classifications merely to pass a test.
- Clearly report surprising or ambiguous classifications, so I can decide whether the classification is reasonable.

REFLECTION
Include a clearly labeled 3-5 sentence reflection section/template in the notebook or README for me to fill in after reviewing real test results. Do not invent what the model classified differently from my expectations.
Explain how LLM-based routing differs from deterministic if-statements in Dana's Bakery, including benefits and tradeoffs in predictability, cost, and testing.

WORKFLOW
1. Briefly explain your intended graph design.
2. Create prompt.md, support_ticket_router.ipynb and optionally README.md.
3. Validate notebook structure and Python syntax locally where possible.
4. Do not claim the OpenAI model ran successfully if no authorized API key was available.
5. Do not commit, push or publish anything yet. We will first run and test the notebook together in Google Colab.

Keep the code as simple and understandable as possible for a student who is learning LangGraph for the first time.

IMPLEMENTATION NOTES: MODEL CHOICE
The single MODEL_NAME constant defaults to gpt-6-luna as requested, matching Homework 1 and the lecturer's example. A short, three-category classification is a focused task, making a low-cost model an appropriate starting point. Official OpenAI documentation lists GPT-6 Luna as an efficient model with structured output support; actual account access and compatibility still need to be tested in Google Colab. Documentation: https://developers.openai.com/api/docs/models/gpt-6-luna
Use Pydantic structured output through ChatOpenAI.with_structured_output with the JSON Schema method, so no external tools or manual category parsing are needed. Set temperature=0 where supported, disable automatic retries, and do not add any model fallback or extra LLM invocation. If the requested model or parameters are rejected, stop with a helpful non-sensitive error and review the configuration explicitly.
The notebook explicitly uses reasoning_effort="none" with temperature=0. OpenAI's parameter guidance says to remove sampling parameters when reasoning effort is not none; the requested model supports none. This keeps the configuration explicit rather than relying on a reasoning default. Documentation: https://developers.openai.com/api/docs/guides/deployment-checklist
