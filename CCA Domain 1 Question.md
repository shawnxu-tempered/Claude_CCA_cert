## 1. Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

A cloud provisioning agent has access to `create_ec2`, `create_rds`, and `create_s3` tools.

The business requires a strict **$1,000 per-session budget limit** across all provisioned resources. The model is currently instructed to track its own spending, but occasionally provisions resources that exceed the budget.

**How should you redesign the architecture to deterministically enforce this financial constraint?**


**A)** Implement a **PreToolCall** hook for all provisioning tools that calculates argument costs against a session budget and blocks any execution if the cumulative total would exceed the **$1,000** limit.

**B)** Fine-tune the model on a dataset of budget-constrained scenarios to teach it to natively track spending across `create_ec2`, `create_rds`, and `create_s3` calls, ensuring it respects the **$1,000** session limit.

**C)** Implement a **PostToolUse** hook that queries the billing API after each resource is created, automatically calling a `terminate_resource` tool if the session's cumulative cost has breached the **$1,000** limit.

**D)** Configure the agent orchestrator's token limit settings to estimate the dollar equivalent of token consumption for each tool call, terminating the session when the cumulative token cost estimate exceeds **$1,000**.


---

Domain： Agentic Architecture & Orchestration
Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

Core: What things should be done by LLm and what things could be done within SDK?
SDK: An orchestration layer to manage the usage of LLm, management of tool uses as a workflow controller.
Hook: customized function to be added within sdk to utilize some user desire. In this problem, the hook is used to calculate the cost of each tool use before letting LLm call the tool. 

Reason: 
A is correct as the hook is triggered before the tool use executes.
B is incorrect as LLm still output the probabilitistic result after fine tuning.
C is incorrect as incorrect as hook is triggered after tool executes which means cost is applied already.
D is incorrect as the true cost is the cost to create AWS resource but not the API cost though claude model.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-5-agent-sdk-hooks

## 2. Subdomain 1.6: Design task decomposition strategies for complex workflows


A development team implemented a per-file analysis agent for code reviews. It successfully catches local syntax errors and style violations. However, it fails to detect when a function signature change in `utils.py` breaks a function call in `main.py`. What architectural addition is required to solve this?



A. Use a single, large prompt containing the full contents of both `utils.py` and `main.py` to provide the agent with enough context to check for call-site errors.

B. Add a cross-file integration pass that takes the outputs of the per-file analyses and the dependency graph to check for boundary issues.


C. Switch the entire review process to a dynamic adaptive decomposition model that re-analyzes file clusters whenever a shared dependency is modified.

D. Combine the contents of `utils.py` and `main.py` into a single temporary file so the existing per-file agent can detect signature mismatches as local errors.


---

Core: Each subagent is working on a pre-file analysis, how to cross-validate them?


Reason: 
A is incorrect as this is not systematically efficient, a larger engineering project will ruin the model context window
B is correct as we have the cross file integration to check what is each agent output and verify if they match with each other.
C is incorrect as incorrect. Even though it sounds reasonable, we can re-design another agent system based on our needs, for example, if we felt main and util are used more relevantly, we just assign one agent to analyze them, but the problem is we were asking what addition to add to solve the problem based on our current pipeline.
D is incorrect as this is not scaleable and also might exceed the context window

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-6-task-decomposition



## 3. Subdomain 1.6: Design task decomposition strategies for complex workflows

An AI agent is tasked with translating a massive 50-chapter fantasy novel. The engineering team notices that translating chapter-by-chapter using a fixed pipeline results in inconsistent character names, shifting tones, and lost global lore. Which architectural pattern should be introduced to resolve these issues?

A. implement a cross-file integration pass that reviews and aligns the local chapter translations against a globally extracted lore glossary.

B. Use dynamic adaptive decomposition to translate chapters out of order based on when characters first appear.

C. Use a map-reduce pattern that translates every paragraph independently and merges them sequentially.

D. Concatenate all 50 chapters into a single prompt to maintain maximum context during translation.

---

Core: Each Chapter is translated on their own, how to cross-validate them so that they are consistent?

Reason: 
A is correct as we added another intergration pass at the end to either use rule-based or LLm based verification on the global consistency for each single outputs regarding chapters.
B is incorrect, it only just change the order character appear, they could also conflict in the end.
C is incorrect as translate them in each paragraph would worsen the consistency.
D is incorrect as this is impractical due to the limit of context window.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-6-task-decomposition


## 4. Subdomain 1.4: Implement multi-step workflows with enforcement and handoff patterns


A financial advisor bot receives a multi-concern request: 'What is my current portfolio balance, and what are the tax implications of selling my Apple stock?' The architect wants to optimize for speed while ensuring accurate, holistic answers. Which approach best satisfies these requirements?


A. Decompose the request into two distinct items, investigate each in parallel using specialized sub-agents that access a shared context (user profile and holdings), and use a synthesizer agent to generate a unified resolution.

B. Route the entire prompt to a tax-specialized agent that internally calls a portfolio-balance API to fetch the current holdings, then uses that data within its tax-estimation logic to produce a combined answer that accounts for cost basis and holding period.

C. Process the request sequentially using a single agent that first retrieves the portfolio balance and then feeds that result into a tax-calculation module, ensuring the tax step has the exact balance figure before computing implications.

D. Reject the prompt and ask the user to submit the portfolio query and the tax query in separate sessions, then process each independently with dedicated agents to maintain deterministic compliance and avoid cross-query contamination.


---

Reason: 

A is correct as Tthis matches Anthropic's recommended orchestrator-worker pattern, where a lead agent decomposes a complex query into subtasks, spawns specialized sub-agents for parallel execution, and synthesizes their findings.

B is incorrect, while this single-agent approach works, it runs sequentially—first fetching balances, then computing taxes—failing to leverage parallel processing. For multi-concern prompt optimized for speed, Anthropic's documentation advises decomposing into independent subtasks that can execute in parallel via sub-agents, then synthesizing results, which reduces total latency compared to a monolithic agent.

C is incorrect as sequential processing ensures data dependencies but sacrifices speed. Anthropic's multi-agent guidance explicitly recommends parallel decomposition for complex tasks to achieve dramatic speedups

D is incorrect as This approach directly contradicts the goal of providing a holistic, single-session answer and degrades user experience.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-4-workflow-enforcement-handoff


## 5. Subdomain 1.4: Implement multi-step workflows with enforcement and handoff patterns

A wealth management bot receives a prompt: 'Analyze my current portfolio risk and suggest three new tech stocks to buy.' The architect decomposes this into a `RiskAnalyzer` agent and a `StockRecommender` agent running in parallel. Both agents need access to the user's current holdings to perform their tasks accurately. How should the shared context be managed?

A. Execute the `RiskAnalyzer` first, append its output to the user's holdings, and pass the combined data to the `StockRecommender` to ensure data consistency.

B. Query the user's holdings twice, once inside each agent's execution loop, to ensure they operate completely independently.

C. Store the user's holdings in a vector database and require both agents to use a RAG tool to retrieve the specific holdings they need.

D. Fetch the user's holdings once in the orchestration layer and inject this data as shared context into the system prompts of both parallel agents before execution.


---

Reason:

A.  This approach converts the workflow into a sequential process, which ignores the architect's requirement for parallel execution. Furthermore, mixing derived analysis with raw input data can confuse downstream agents and make the system harder to debug.

B. Querying the user's holdings twice is redundant and inefficient, increasing latency and database load. It also introduces the risk of data inconsistency if the holdings change between the two independent fetches.

C. RAG and vector databases are designed for unstructured data retrieval. Using them for structured user holdings adds unnecessary complexity, potential retrieval errors, and latency compared to simply injecting the data into the context.

D. is correct. Fetching the data once at the orchestration layer and injecting it into both parallel agents is the most efficient and reliable pattern. It ensures both agents work from an identical snapshot of the user's data, which is essential for consistent multi-step task execution.


Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-4-workflow-enforcement-handoff


## 6. Subdomain 1.6: Design task decomposition strategies for complex workflows

A legal agent is analyzing 500 existing patents to find prior art that invalidates a new invention claim. The system currently uses per-file local analysis to check each patent individually against the new claim. However, the legal team notes that invalidation often requires combining the claims of *two different* existing patents. What architectural addition is needed?

A) An adaptive investigation plan that generates new patent claims based on the local analysis.

B) A cross-file integration pass that evaluates combinations of the extracted claims from the local passes.

C) A fixed prompt chain that reads all 500 patents sequentially in a single context window.

D) Dynamic adaptive decomposition to autonomously search the internet for additional prior art.


---

Core: Integration of outputs from different subagents. Similar as Question 1

B is correct as the Claude Certified Architect – Foundations exam guide explicitly recommends splitting large code reviews into per-file local analysis passes plus a separate cross-file integration pass. This approach is directly applicable here: after extracting claims from each patent locally, a cross-file integration pass can evaluate combinations of those claims to identify prior art that requires multiple patents.

Refer:  https://claudecertificationguide.com/learn/1-agentic-architecture/1-6-task-decomposition


## 7. Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

A marketing agent uses a `send_sms` tool connected to a third-party API. The API strictly limits requests to 10 per minute. If the agent exceeds this, the API bans the account for 24 hours. The agent sometimes generates a loop of rapid tool calls, risking a ban. How can you programmatically protect the API account using Anthropic's tool use features?

A) Implement a PreToolCall hook that checks a local token bucket rate limiter. If the limit is reached, the hook blocks the tool's execution and injects a message into the conversation telling the model to wait.

B) Implement a PostToolCall hook that inspects the API response for rate-limit headers and, when a warning is present, injects a message instructing the model to pause further send_sms calls for 60 seconds.

C) Configure the orchestrator to automatically terminate the agent session when it detects more than 10 send_sms calls within a single minute. This prevents the agent from exceeding the API's rate limit by stopping the session before a ban occurs.

D) Add a system prompt instruction that directs the model to track its own send_sms calls and not exceed 10 per minute. The prompt includes a rule to wait until a full minute has passed since the first call before sending additional messages.


---

Core: Insert an pretool Hook calling before we call the model for tool use within SDK

A is correct: `PreToolCall` hook intercepts tool calls before execution, allowing a token bucket rate limiter to block the call if the limit is reached. This proactive approach prevents the 11th call from ever being made, avoiding the API ban. Injecting a message instructing the model to wait provides a graceful, programmatic safeguard.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-5-agent-sdk-hooks



## 8. Subdomain 1.3: Configure subagent invocation, context passing, and spawning

A market research platform needs to analyze competitor pricing across five different e-commerce websites simultaneously to reduce overall system latency. Currently, a coordinator agent analyzes one website, waits for the result, and then moves to the next. How can the architect redesign the system to execute these tasks concurrently using an agentic framework?

A) Modify the coordinator's system prompt to instruct it to emit multiple, independent Task tool calls within a single response turn, with each call targeting one of the five websites.

B) Update the AgentDefinition of the subagent to set execution_mode: parallel and pass an array of five URLs in a single Task tool call, so the subagent processes all sites concurrently.

C) Configure the coordinator to use the Message Batches API to submit five separate analysis requests asynchronously, then collect the results once all batches complete.

D) Use fork-based session management to branch the coordinator's state into five parallel threads before invoking the Task tool, allowing each thread to independently analyze one website.


---


A is correct as By instructing the coordinator to emit multiple independent `Task` calls in one turn, the orchestrator can spawn subagents that operate in parallel, significantly reducing latency.

B is incorrect as Anthropic's agentic framework does not support an `execution_mode: parallel` setting on `AgentDefinition`

C is incorrect as Anthropic does not offer a `Message Batches API` for asynchronous request submission.

D is incorrect as Concurrency is achieved through the coordinator spawning separate subagents within the same session, not by branching the coordinator's state.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-3-subagent-invocation-context


## 9. Subdomain 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns

A multi-agent system generates personalized marketing campaigns. The Coordinator invokes a `ProfileAgent` (which returns user demographics as a nested JSON object) and a `ProductAgent` (which returns product specs as a JSON array). Currently, the Coordinator takes these raw JSON outputs and dumps them directly into the prompt of a `CopywriterAgent`. The `CopywriterAgent` frequently gets confused by the database schemas and accidentally includes internal JSON keys (e.g., "user_id_fk": 992) in the final marketing emails. How should the Coordinator's information routing responsibilities be improved?


A) The Coordinator should bypass the CopywriterAgent and use a deterministic Python script to directly map JSON values, such as user demographics, into placeholders within a pre-approved HTML email template.

B) The Coordinator should parse and filter the raw JSON from data agents, then translate the results into a clean, natural language context before passing it to the CopywriterAgent.

C) The ProfileAgent and ProductAgent should be reconfigured to communicate directly with the CopywriterAgent, bypassing the Coordinator and exposing their data via a shared GraphQL endpoint for direct queries.

D) The Coordinator should instruct the CopywriterAgent to first write and execute a Python script to parse the raw JSON payloads, extract relevant fields, and then use that structured data to generate the final email copy.


---
A is incorrect as While deterministic scripts provide high reliability for rigid templates, this solution abandons the agentic design.

B is correct as: In a well-architected Coordinator-Subagent pattern, the Coordinator acts as a mediator and abstraction layer. It should parse and filter the raw JSON from data agents, then translate the results into a clean, natural language context before passing it to the CopywriterAgent. Since LLm is good to understand Natural langauge.

C is incorrect as Direct communication between subagents violates the hub-and-spoke model of the Coordinator pattern.

D is incorrect as Forcing the CopywriterAgent to handle raw JSON parsing via code increases the token cost, latency, and the likelihood of reasoning errors.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-2-orchestration-patterns


## 10. Subdomain 1.7: Manage session state, resumption, and forking

A technical writer is using Claude to draft a comprehensive 50-page user manual for a proprietary software platform. The session has been active for three days, and Claude has successfully ingested the platform's internal wiki and drafted the first 25 pages. The writer goes on a two-week vacation. During this time, the software platform undergoes absolutely no changes. Upon returning, what is the most efficient way for the writer to continue drafting the manual?


A) Start a new session and provide a structured summary of the first 25 pages, because sessions automatically expire and lose context after 7 days of inactivity.

B) Use --resume to continue the existing session, as the underlying platform has not changed and the prior context remains entirely valid for completing the manual.

C) Start a new session, re-upload the internal wiki, and ask Claude to analyze the first 25 pages to rebuild its context window from scratch.

D) Use fork_session to create a new branch for the remaining 25 pages, as appending to a session older than a week causes severe latency degradation.


---

Fork session: In this problem, Fork session is more like the Git Branch, for example, if we have 2 different methods starting from some point and we would like to try different approaches, we can start a fork session to separate from the original "branch".

Original Session
        │
        │
   fork_session
        │
   ┌────┴────┐
   │                │
Original     Fork

A is incorrect because session state persistence in professional orchestration environments does not have a mandatory 7-day expiration that forces a transition to summaries. Furthermore, providing a summary is less efficient and less accurate than maintaining the original full-text context.

B is correct as Resuming a session is the most efficient approach because it leverages the existing context window. Since the underlying source material (the wiki) and the progress (the first 25 pages) have not changed, resuming allows Claude to continue with the full history of the session without the need for redundant data ingestion or reprocessing.

C is incorrect as This is the most inefficient method. Re-uploading the wiki and re-analyzing the existing 25 pages wastes significant tokens and time.

D is incorrect as Forking is typically used to branch a conversation into different directions or to test alternate completions from a specific checkpoint. It is not necessary for simple resumption.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-7-session-state-resumption


## 11. Subdomain 1.6: Design task decomposition strategies for complex workflows

A legal tech company is building a workflow to analyze 200-page merger agreements. The system must extract defined terms, verify their correct usage throughout the document, and flag inconsistencies. When attempting this in a single prompt, the model misses many inconsistencies. Which prompt chaining pattern is the most effective and recommended approach to resolve this issue?

A) A parallel processing chain where one agent extracts defined terms while a second agent simultaneously scans the document to flag usage inconsistencies.

B) A fixed prompt chain that first extracts defined terms, passes them to a second prompt to analyze usage, and a third to flag inconsistencies.

C) A map-reduce chain that splits the document into 10-page chunks, extracts defined terms from each chunk, and then merges the results to flag inconsistencies.

D) A dynamic decomposition strategy where an agent autonomously identifies pages with defined terms, extracts them, and then verifies their usage across the document.


---

B is correct, the question askes what is the most efficient and recommended prompt chaining pattern. According to Anthropic's documentation, breaking a complex task into a sequence of simpler, focused subtasks is a recommended best practice known as 'prompt chaining' or 'multi-stage prompting'. This approach improves accuracy and consistency by allowing the model to focus on one subtask at a time, using the output of one step as the input for the next.



Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-6-task-decomposition

## 12. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

Claude returns a `stop_reason` of `tool_use` with three parallel tool calls: `get_weather`, `get_traffic`, and `get_calendar`. The application executes them concurrently. `get_weather` and `get_calendar` succeed, but `get_traffic` times out and throws an exception. How should the architect construct the next request in the agentic loop to ensure Claude can handle the partial failure and continue reasoning?

A) Send the two successful tool_result blocks in a user message, then wait for Claude to explicitly request the get_traffic tool again in a subsequent turn.

B) Terminate the loop immediately, as the Anthropic API requires all parallel tool calls to succeed before the conversation history can be appended.

C) Append a user message containing only the two successful tool_result blocks, omitting the failed tool entirely so Claude does not get confused by the timeout.

D) Append a single user message containing three tool_result blocks: two with the successful data, and one containing the error message with is_error: true for the failed tool.


---
D is correct as In parallel tool use scenarios, the next turn must include a `tool_result` block for every `tool_use` ID generated by the model. Including all three blocks, with the failed one marked with `is_error: true`, ensures the conversation history remains consistent. This allows Claude to incorporate the successful data while reasoning about the failure to decide on the next step.


Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-1-agentic-loops


## 13. Subdomain 1.3: Configure subagent invocation, context passing, and spawning

A travel planning application uses a 'FlightSearch' subagent and a 'HotelSearch' subagent. The coordinator needs to invoke the HotelSearch subagent after the FlightSearch subagent successfully finds a flight. During testing, the HotelSearch subagent keeps searching for hotels on the wrong dates. How should the architect ensure the HotelSearch subagent receives the correct temporal context?

A) Add the FlightSearch tool to the HotelSearch subagent's allowedTools so it can independently verify the dates itself by querying the flight search results directly, bypassing the need for the coordinator to pass the dates.

B) Configure the HotelSearch subagent's AgentDefinition to inherit the FlightSearch subagent's memory state, so the hotel search automatically uses the flight's arrival and departure dates from the shared context.

C) Ensure the coordinator explicitly extracts the arrival and departure dates from the FlightSearch results and includes them in the prompt when invoking the HotelSearch subagent via the Task tool.

D) Use fork-based session management to merge the FlightSearch and HotelSearch threads before generating the final itinerary, allowing the hotel search to access the flight search's temporal context through a shared session state.


---
Core: There is no sharing info among/between subagents! Anthropic suggests to use the coordinator to manage the info and pass to subagents.


C is correct as : The coordinator should explicitly extract the arrival and departure dates from the FlightSearch results and include them in the prompt when invoking the HotelSearch subagent. This ensures a deterministic, auditable handoff and prevents the subagent from using wrong dates.



## 14. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

A cybersecurity agent uses a `scan_port` tool. The developer executes the tool and appends the result as a plain text string to the end of the last `user` message, rather than using a structured `tool_result` block. During testing, Claude begins hallucinating tool outputs and loses track of which ports it has already scanned. Why does this implementation cause model degradation?


A) Appending plain text instead of a structured tool_result block breaks the explicit linkage between the assistant's tool_use_id and the result, degrading the model's ability to reason about the action.

B) The developer must use a system message to inject plain text results, as user messages are strictly reserved for human input and cannot contain tool data without corrupting the conversation context.

C) Plain text strings bypass the model's internal safety filters, causing it to hallucinate malicious port activity instead of reading the actual scan results, which undermines the agent's situational awareness.

D) Plain text strings do not support the is_error flag, so the model cannot distinguish between open and closed ports, leading it to misinterpret every scan result as a successful connection.


---

**Correct workflow:** When Claude returns a `tool_use` response, the application executes the requested tool(s) externally. After execution, the application must append a new **user** message containing a structured `tool_result` block for **each** `tool_use`, with every `tool_result` explicitly linked to its corresponding `tool_use_id`. Claude then receives these structured tool results as observations and continues its reasoning based on the updated conversation state.

**Why the incorrect implementation causes degradation:** Instead of returning structured `tool_result` blocks, the developer appends the tool output as plain text to the previous user message. This breaks the explicit association between each `tool_use_id` and its corresponding result, so Claude can no longer reliably determine which tool call produced which output. As the conversation grows or multiple tool calls are involved, the model loses track of the execution state, may hallucinate missing results, repeat tool calls unnecessarily, or make incorrect assumptions because the expected tool-use protocol has been violated.


A is correct as： Claude's Messages API requires a structured action-observation loop where a `tool_use` (from the assistant) is followed by a `tool_result` (from the user). The `tool_result` block MUST contain a `tool_use_id` that matches the assistant's request. This linkage provides provenance and allows the model to reliably map observations back to specific actions. Without it, the model loses the logical thread, leading to hallucinations and a failure to maintain accurate state.


Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-1-agentic-loops



## 15. Subdomain 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns


A legal tech application features a conversational interface managed by a Coordinator agent. When a user asks, "Can you summarize the liability clause from that last vendor contract?", the Coordinator correctly identifies the intent and delegates the task to a specialized ContractAnalyzer subagent. However, the ContractAnalyzer responds with an error, stating it does not know which contract the user is referring to. What is the architectural root cause of this failure?

A) The Coordinator's system prompt lacks instructions for the subagent to access the global session state, which holds the reference to the "last vendor contract."

B) Subagents operate with isolated context; the Coordinator failed to explicitly pass the relevant document and conversation history in its delegation prompt.

C) The multi-agent system is utilizing an outdated API version that lacks support for automatic context inheritance, preventing the Coordinator from passing session data.

D) The ContractAnalyzer's temperature setting is too low, preventing it from inferring the implicit context of the conversation to identify the specific vendor contract.


---
Core: In coordinator-subagent architectures, Coordinator is responsible to provide relevant info for the subagents to address their subtasks.


B is correct: In coordinator-subagent architectures, subagents typically operate with isolated contexts. The Coordinator is responsible for gathering relevant conversation history and document references and explicitly including them in the delegation prompt, which it failed to do here.


Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-2-orchestration-patterns


## 16. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

A developer is implementing an agentic loop for a customer service bot. When the API returns a `stop_reason` of `tool_use`, the application executes the tool successfully. To continue the autonomous execution and allow Claude to reason about the next action, how must the developer pass the result back to the model?

A) Replace the original tool_use block in the assistant's last message with the tool's output. Then, resend the entire modified messages array to the API.

B) Send a new request to the API with the messages array containing the previous history plus a new user message containing the tool_result block.

C) Call the API's dedicated /v1/messages/tool_callback endpoint. The request body should contain the tool_use_id and the tool's JSON output to continue the agentic loop.

D) Append the tool result to the system prompt and initiate a new conversation, sending a request with the updated prompt to ensure the model has the tool's output.


---
pipeline：

                    ┌──────────────────────────────┐
                    │            User              │
                    │ "Book me a hotel in Tokyo." │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │           Claude             │
                    │                              │
                    │ Think...                     │
                    │                              │
                    │ I need a tool.               │
                    │                              │
                    │ tool_use(FlightSearch)       │
                    └──────────────┬───────────────┘
                                   │
                     stop_reason = tool_use
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │        Application           │
                    │ (Your Python/Backend Code)   │
                    │                              │
                    │ Executes FlightSearch()      │
                    │                              │
                    │ Gets:                        │
                    │ Arrival: Jul 20             │
                    │ Departure: Jul 26           │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
      Append a NEW user message containing a structured tool_result
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ role = "user"                │
                    │                              │
                    │ tool_result                  │
                    │ tool_use_id = xxx            │
                    │ Arrival: Jul 20             │
                    │ Departure: Jul 26           │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │           Claude             │
                    │                              │
                    │ Receives the observation     │
                    │                              │
                    │ Thinks again                 │
                    │                              │
                    │ "Now I should search hotels" │
                    │                              │
                    │ tool_use(HotelSearch)        │
                    └──────────────────────────────┘
A is wrong because we never replace the tool use block in the user message prompt.

B is correct since： This is the documented procedure for a client-driven agentic loop. According to Anthropic's official documentation, when a `tool_use` stop reason occurs, the application must send a new request containing the entire previous conversation history, the assistant's message with the `tool_use` block, and a new `user` message that contains the corresponding `tool_result` block. This allows Claude to maintain context and reason about the next step.

C is wrong because Claude does not provide tool callback endpoint. All interactions, including providing tool results, are handled by sending requests to the main `/v1/messages` endpoint with a properly structured `messages` array.

D is wrong because:  The `system` prompt is meant for high-level, persistent instructions, not for transactional data like tool results.


Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-1-agentic-loops


## 17. Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

An enterprise agent aggregates customer feedback from three distinct platforms using separate MCP tools. The Zendesk tool returns sentiment as a float between -1.0 and 1.0, the App Store tool returns an integer from 1 to 5, and the Twitter tool returns categorical strings ('positive', 'neutral', 'negative'). The model frequently hallucinates when asked to calculate an overall customer satisfaction score due to the mixed data types. What is the most robust architectural approach to resolve this?

A) Implement a PreToolCall hook on all three tools that injects a requested_format="1-100" parameter into the outgoing API requests.

B) Implement a PostToolUse hook that intercepts the responses from all three tools, maps the disparate values to a standardized 1-100 integer scale using custom logic, and returns the normalized schema to the model.

C) Create a separate normalize_sentiment tool and instruct the model to call it sequentially after receiving data from any of the feedback platforms.

D) Update the system prompt with a detailed conversion matrix, instructing the model to mathematically convert all scores to a 1-100 scale before performing any aggregations.


---

B is correct since A PostToolUse hook is the most robust location to intercept tool responses before they are returned to the model. By using custom logic in this hook, you can deterministically normalize the mixed data types (floats, integers, and categories) into a single canonical 1-100 scale. This ensures the model receives a consistent schema, significantly reducing cognitive load and the risk of hallucination.

C is wrong: Creating a separate normalization tool increases latency and complexity by requiring multiple tool calls. It relies on the model to correctly orchestrate the sequence, which is less reliable than automatic interception and normalization at the SDK level.

In this case, rather than adding anther subagent to complete this task, a simple posttooluse hook is a better choice.

## 18. Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

A procurement agent aggregates component prices from three regional suppliers (US, EU, JP) using three different MCP tools. The tools return prices in USD, EUR, and JPY respectively. The model struggles to accurately identify the cheapest supplier due to fluctuating exchange rates. What is the best architectural pattern to solve this?

A) Fine-tune the model on historical exchange rate data so it can perform the conversions natively in its latent space, allowing it to estimate normalized prices without calling any external conversion tool.

B) Implement a PreToolCall hook that injects currency="USD" into the arguments of the EU and JP supplier tools, forcing the tools to return prices in USD so the model can compare them directly.

C) Instruct the model to call a get_exchange_rate tool before any price comparison, then use the returned rates to convert all supplier prices to a common currency within the agent's reasoning step.

D) Implement a PostToolUse hook on all three supplier tools that intercepts the response, fetches the live exchange rate, converts the price to a base currency (e.g., USD), and appends the normalized price to the tool result.


---
Core: This is a very similar question as the previous Question 17


D is correct.


## 19. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

A developer is building an autonomous research agent using Claude. To determine when the agent has finished its research, the developer's code checks if the assistant's response contains the phrase 'Research complete:'. If this phrase is found, the agent's main loop terminates. What is the architectural assessment of this approach?

A) This is a best practice when the system prompt enforces the exact phrasing of 'Research complete:' using XML tags, and the agent's main loop parses the response for that tag to terminate deterministically.

B) This is an anti-pattern because it relies on parsing non-deterministic natural language for control flow. The developer should instead instruct the model to call a specific tool, such as research_complete(), to signal task completion.

C) This is an anti-pattern; the developer should instead check for a specific stop_sequence like </research_complete> in the API response, and configure the agent to terminate when that sequence is detected.

D) This is an anti-pattern; the developer should instead evaluate if stop_reason equals end_turn to safely and deterministically terminate the loop, avoiding reliance on natural language parsing. By checking the API response's stop_reason field.


---

D is correct： The `stop_reason` field provides a deterministic, language‑agnostic mechanism to know why the model stopped generating. In an agent loop, when the model has completed its task and has no further tool calls, `stop_reason` is set to `end_turn`. This offers a reliable programmatic signal for loop termination, eliminating the anti‑pattern of parsing the assistant’s text. This style is demonstrated in Anthropic’s official API documentation and is the behavior used by Claude Code, where the execution loop naturally terminates when the model produces a plain‑text response without tool invocations.

Remember, Do not fully trust the returned natural langauge, Claude returns structure text with `stop_reason` which is deterministic.

Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-1-agentic-loops


## 20. Subdomain 1.3: Configure subagent invocation, context passing, and spawning

A financial analysis system extracts quarterly earnings data using a 'DataMiner' subagent and generates a chart using a 'Grapher' subagent. When the DataMiner passes its findings back to the coordinator as a conversational summary, and the coordinator passes that summary to the Grapher, the Grapher frequently plots incorrect numbers or mislabels axes. What is the most robust architectural solution to this context passing issue?

A) Configure the Grapher subagent's AgentDefinition to use a lower temperature setting so that it relies more heavily on the coordinator's prompt and reduces spontaneous numerical variations during chart generation.

B) Add a DataValidation tool to the coordinator that compares the Grapher's output against the DataMiner's summary and flags discrepancies in axis labels or plotted numbers before the chart is presented to the user.

C) Instruct the DataMiner to output its findings as a structured CSV or JSON array, and have the coordinator pass this exact structured output in the prompt to the Grapher subagent.

D) Use fork-based session management to allow the Grapher to iteratively query the DataMiner in a private thread, replotting the chart each time until the plotted values match the expected quarterly earnings data.


---


principal： Avoiding direct communication between subagents, the ideal case is sending structured result from subagent to coordinator agent, and coordinator send structured prompt for Grapher subagent.

C is correct. Instructing the DataMiner to output its findings as a structured CSV or JSON array ensures that exact, machine-readable numeric and field semantics are preserved. This eliminates the ambiguity introduced by free-form conversational summaries, allowing the Grapher to process concrete values and labels directly. This is the most robust way to maintain data integrity across subagent boundaries.


Refer: https://claudecertificationguide.com/learn/1-agentic-architecture/1-3-subagent-invocation-context


## 21. Subdomain 1.7: Manage session state, resumption, and forking

In an agentic architecture using Claude, what is a recommended practice for managing session state and resuming a conversation when built-in session persistence is not available?

A) Save the terminal output to a text file and upload it to a new Claude session later.

B) Start the session initially with a named identifier and use `--resume <session-name>` when returning to continue the specific prior conversation.

C) Ask Claude to generate a structured summary, then paste that summary into a new session.

D) Use the `fork_session` command before closing the terminal to ensure the state is saved in a parallel branch.


---

C is correct as: Anthropic's official documentation and best practices recommend asking Claude to generate a structured summary (a 'handoff document') of the current conversation, including what was done, what worked, and next steps. This summary can then be used as the initial context for a new session, providing continuity while ensuring a clean and focused context. This is especially useful when output quality begins to degrade due to a large context window.

B and D are wrong because both rely on the built in seesion persistence.

## 22. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

An agentic loop includes a `capture_screenshot` tool. When Claude calls this tool, the application successfully captures a base64-encoded PNG image of the current web page. The developer needs to pass this image back to Claude so it can analyze the UI and decide on the next action. How should the developer construct the `tool_result` block to include the image?

A) Convert the base64 image to a URL and pass the URL as a plain text string inside the tool_result block.

B) Include an image content block containing the base64 data and media type within the content array of the tool_result block.

C) Images cannot be passed inside tool_result blocks; the developer must restart the conversation and provide the image in the initial prompt.

D) Append the image as a separate user message immediately following the user message that contains the tool_result block.


---

B is correct. In the Anthropic Messages API, the 'tool_result' block contains a 'content' field that accepts an array of content blocks. This array can include 'text' blocks and 'image' blocks (containing base64 data and the media type), allowing Claude to receive and analyze the visual output of the screenshot tool.

D is wrong as: Tool results must be returned in a message with the 'user' role that specifically includes a tool_result block corresponding to the model's tool_use request. Appending a separate user message breaks the specific tool-calling handshake protocol.

A is wrong because: Passing a URL as a plain text string does not allow Claude to process the image data visually.


## 23. Subdomain 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns

A financial firm uses a Coordinator agent to interact with users about uploaded 100-page annual reports. When a user asks for a risk analysis, the Coordinator delegates to a RiskSubagent. To save tokens, the developer configures the Coordinator to only send the user's specific question to the RiskSubagent, assuming the subagent can access the uploaded document from the session state. The subagent consistently returns generic, unhelpful answers. What must the developer change?


A) Upgrade the RiskSubagent to a more advanced model tier with a larger context window so it can automatically infer the necessary document details from the shared session state.

B) Configure the RiskSubagent with a web search tool. The subagent should then be prompted to extract the company name from the user's query to find and analyze the report online.

C) Explicitly extract and pass the relevant chunks of the annual report from the Coordinator to the RiskSubagent, as subagents operate with isolated context.

D) Remove the Coordinator agent entirely. This change allows the user to chat directly with the RiskSubagent, which can then access the full document from the shared session state.


---


C is correct as In a coordinator-subagent architecture, subagents generally operate with isolated context windows. Each API call to a subagent must include the relevant information needed for that specific task. By extracting and passing specific chunks (a RAG approach), the developer ensures the subagent has the necessary context to perform the risk analysis while still maintaining token efficiency compared to sending the full 100-page report.

B is incorrect as Web search is an unreliable and inefficient substitute for the document already provided to the system. It risks retrieving incorrect versions, may fail for private/internal documents, and introduces unnecessary latency and privacy concerns.

A is unnesseccary as Upgrading the model tier may improve general reasoning, but it does not fix the root cause of the failure.


## 24. Subdomain 1.7: Manage session state, resumption, and forking

An infrastructure engineer uses Claude to write a Terraform script that provisions a complex multi-region AWS environment. Claude executes the `terraform apply` command via a tool, which takes 45 minutes to complete. The engineer closes the session. An hour later, the infrastructure is fully provisioned, but several AWS ARNs generated during the process need to be mapped into an application configuration file. What is the most efficient way to use Claude for this mapping task?

A) Use `fork_session` from the moment before `terraform apply` was executed, and ask Claude to predict the ARNs based on the Terraform plan.

B) Start a new session with a structured summary of the goal and provide the newly generated Terraform state file or outputs, as the previous session's tool results are stale.

C) Resume the previous session, as Claude's context window will automatically poll the AWS API to update the ARNs once the session is reawakened.

D) Resume the previous session and ask Claude to read the terminal history to find the ARNs, as tool execution outputs are always perfectly preserved across session closures.


---

B is correct： Starting a new session with a concise summary and providing the current source of truth (Terraform state or outputs) is the most efficient and reliable method. Because the previous session's tool outputs may be outdated, truncated, or difficult to retrieve after a long delay and session closure, providing the actual generated state ensures Claude has the most accurate information.

C is wrong： because Claude does not have the capability to automatically poll external APIs like AWS or refresh infrastructure state on its own when a session is resumed. External state changes must be provided to the model manually or via a tool call.

## 25. Subdomain 1.4: Implement multi-step workflows with enforcement and handoff patterns

A travel booking bot using Claude is processing a complex flight cancellation. Mid-process, the airline API returns a 'manual intervention required' error. The bot must escalate to a human agent immediately, but the human agents' CRM system does not support importing chat transcripts. How should the architect design this mid-process escalation?

A) Trigger a structured handoff protocol where Claude requests a `compile_handoff` tool, and the application executes it to generate a summary containing the customer ID, root cause, and recommended action to populate the CRM.

B) Send the raw JSON array of the conversation history to the CRM so the human agent can parse the tool calls and API errors manually.

C) Prompt the bot to ask the user to write a brief summary of the steps they just performed before triggering the escalation webhook.

D) Programmatically extract all user messages from the session state using `get_session_messages()`, concatenate them into a single text block, and send it to the CRM.


---

A is correct, claude call the tool use and the pre-defined app start to execute the handoff process.



## 26. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

An agentic loop is executing a long-running data analysis task. After the agent calls the fetch_data tool, the application detects that the user has clicked a "Cancel" button on the frontend. The developer wants to gracefully interrupt the agent's current plan and instruct it to summarize what it has found so far, rather than continuing to analyze. How should the architect inject this interruption instruction into the active agentic loop?

A) Replace the tool_result block entirely with a text block containing the instruction, as sending a tool_result will force the model to continue its original plan.

B) Call the /v1/messages/cancel endpoint to terminate the current loop, then start a completely new conversation thread asking for a summary.

C) Include a text content block containing the interruption instruction alongside the tool_result block within the next user message.

D) Modify the stop_reason of the previous API response to user_cancelled before appending it to the conversation history.


---

C is correct as This is the standard and most effective pattern for human-in-the-loop intervention in Anthropic agentic workflows. By providing both the tool_result (to satisfy the API requirements and provide the data) and a text block (containing the new 'summarize' instruction) in the same user turn, the architect can pivot the model's behavior while maintaining the full context of the session.

B is wrong as： There is no standard /v1/messages/cancel API endpoint that handles loop redirection.

D is wrong as： The stop_reason is a read-only metadata field returned by the Claude API (e.g., end_turn, tool_use, max_tokens) to explain why generation stopped. It is not an input parameter for the Messages API, and modifying it in the message history will not influence the model's future logic or handle a user interruption.


## 27. Subdomain 1.4: Implement multi-step workflows with enforcement and handoff patterns

A telecommunications customer asks a support bot: 'Why is my bill so high this month, and can I upgrade to the premium plan?' To ensure low latency and high accuracy, how should the architect design the workflow for this multi-concern request?

A) Prompt the main agent to use the check_bill and upgrade_plan tools in a single turn, relying on the LLM's internal parallel tool calling capabilities without shared context.

B) Process the request sequentially: first resolve the billing issue, then handle the upgrade, to avoid confusing the model's context window.

C) Use a router to decompose the request into 'billing' and 'upgrade' tasks, dispatch them to parallel sub-agents with shared user context, and use a synthesizer agent to merge the results.

D) Send the raw prompt to two separate agents simultaneously and concatenate their raw text responses with a newline character before sending to the user.


---

Core: The most efficient way and most accurate way is to use a main agent to assign small tasks for subagent, and they run in parallel.

C is correct: The router-orchestrator pattern is the most effective for multi-intent requests. Decomposing the prompt into specialized tasks allows for parallel execution (minimizing latency) and high domain-specific accuracy. Using shared context and a synthesizer ensures the final response is coherent, consistent, and addresses all user concerns accurately.


## 28. Subdomain 1.6: Design task decomposition strategies for complex workflows

A media company uses an AI agent to review 3-hour video transcripts for brand safety, narrative consistency, and pacing. Initially, the entire transcript was processed in one prompt, but the model hallucinated events due to attention dilution. The architect split the workflow into 10-minute local analysis chunks. Now, the agent accurately flags local brand safety issues but completely misses narrative inconsistencies that span across the entire video. How should the architect resolve this?

A) Revert to a single prompt but use strict XML tags to separate the 10-minute chunks within the context window.

B) Switch the entire workflow to dynamic adaptive decomposition so the agent can autonomously decide which 10-minute chunks to read.

C) Increase the model's temperature and use a map-reduce pattern to process all chunks in parallel without a final integration step.

D) Implement a cross-file integration pass that analyzes the summarized narrative outputs from the local chunk passes to evaluate global consistency.


---

D is correct to integral local chunk results to ensure global consistency.


## 29. Subdomain 1.3: Configure subagent invocation, context passing, and spawning

A creative writing application generates a comprehensive baseline plot outline. From this single outline, the system needs to generate three distinct story endings (optimistic, pessimistic, and ambiguous). The architect wants to ensure the generations do not influence each other, while avoiding the latency and cost of having the model re-read and re-process the baseline outline from scratch for each ending. Which approach best satisfies these requirements?


A) Use fork-based session management to create three separate branches from the conversation state immediately following the baseline outline generation.

B) Spawn three parallel subagents using the Task tool, passing the baseline outline in the prompt of each subagent.

C) Define three separate AgentDefinition configurations with different system prompts and invoke them sequentially in the same session.

D) Configure the coordinator to emit three sequential Task tool calls, clearing the context window between each call to prevent cross-contamination.


---

A is correct： Anthropic’s fork-based session management (e.g., via `/branch` in Claude Code or the `--fork-session` CLI flag) copies the entire conversation history up to the baseline outline into three independent branches. Because the shared prefix is cached by the API, subsequent generations in each branch reuse the cached baseline tokens, reducing both latency (near-instant) and cost (cached tokens are ~90% cheaper). This approach perfectly satisfies the requirements of independence and efficiency.


## 30. Subdomain 1.6: Design task decomposition strategies for complex workflows

A healthcare platform processes standard patient intake forms to extract demographic data, and then investigates complex, open-ended medical histories to suggest potential clinical trials. The engineering team needs to design the orchestration layer for these two distinct tasks. Which combination of decomposition patterns is most appropriate?

A) Dynamic adaptive decomposition for the intake forms, and a fixed sequential pipeline for the clinical trial investigation.

B) Fixed sequential pipeline for both the intake forms and the clinical trial investigation.

C) A fixed sequential pipeline for the intake forms, and dynamic adaptive decomposition for the clinical trial investigation.

D) Dynamic adaptive decomposition for both the intake forms and the clinical trial investigation.


---

C is correct： This approach matches the orchestration complexity to the task requirements. A fixed sequential pipeline provides reliability and efficiency for the predictable, structured extraction of demographic data. Conversely, dynamic adaptive decomposition allows the agent to iteratively explore complex, unstructured medical histories and synthesize trial suggestions based on context-specific reasoning.


## 31. Subdomain 1.7: Manage session state, resumption, and forking

A systems administrator is using Claude to analyze a massive `access.log` file to track down the source of a DDoS attack. Claude has read several chunks of the file and identified a suspicious IP range. At midnight, the server's log rotation daemon archives `access.log` to `access.log.1` and creates a fresh, empty `access.log`. The sysadmin wants to continue the investigation the next morning. How should the sysadmin proceed with the session?

A) Resume the session and explicitly inform Claude about the log rotation, directing it to analyze `access.log.1` instead of the now-empty `access.log`.

B) Resume the session and ask Claude to continue reading `access.log`; the agent will automatically realize the file is empty and search the directory for the archive.

C) Start a new session, because external log rotation permanently corrupts the file pointers in Claude's existing context window.

D) Use `fork_session` to branch the state, which automatically detects the inode change and redirects Claude's file-reading tools to the archived log.


---
A is correct： Resuming the existing session allows Claude to retain its prior reasoning and findings (such as the identified IP range). Since Claude cannot independently track OS-level filesystem changes like log rotation, the user must explicitly provide the new path (access.log.1) to ensure continuity without losing progress.

Because Claude does not track down the system log file, he still think the previous one is the original one.


## 32. Subdomain 1.6: Design task decomposition strategies for complex workflows

An autonomous red-teaming agent is given a target IP address to perform a penetration test. The agent's next actions are completely unknown until the initial port scan returns results (e.g., finding an open FTP port requires different tools than finding an exposed web API). Which decomposition strategy is required for this agent?

A) First mapping the network structure, then using a fixed prompt chain.

B) A cross-file integration pass of all historical scan logs.

C) Dynamic adaptive decomposition based on intermediate findings.

D) A fixed sequential pipeline of all known exploits.


---

C is correct： Dynamic adaptive decomposition is the appropriate strategy for situations where intermediate findings determine the next subtask and tool choice. This allows the agent to inspect real-time scan results and then decide whether to pursue specific service workflows (like FTP or web APIs), ensuring flexibility and responsiveness to discovered conditions.


## 33. Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

A supply chain agent receives automated inventory alerts via a `get_alerts` tool. The tool's API only returns raw SKU codes (e.g., 'SKU-992'). The model needs the human-readable product name and category to draft meaningful emails to suppliers. Prompting the model to sequentially call a `lookup_sku` tool for every alert introduces unacceptable latency and token costs. What is the most efficient architectural solution?

A) Implement a PostToolUse hook on `get_alerts` that intercepts the raw SKU array, queries a fast in-memory cache for the product details, and appends the enriched data to the tool result before the model processes it.

B) Implement a PreToolCall hook on `get_alerts` that injects a `include_metadata=true` parameter, assuming the external API will automatically adapt to the new parameter.

C) Combine the `get_alerts` and `lookup_sku` tools into a single prompt instruction, forcing the model to execute them in parallel.

D) Fine-tune the model on the company's entire product catalog so it can natively map SKU codes to product names from memory.


---

A is correct, again, trigger and postTooluse took is always efficient and cheap than another prompt for easy task.


## 34. Subdomain 1.1: Design and implement agentic loops for autonomous task execution

Claude returns a `stop_reason` of `tool_use` with three `tool_use` blocks in a single response. The developer executes all three tools successfully. To append the results to the conversation history, the developer creates three separate `user` messages, each containing one `tool_result` block, and appends them sequentially to the `messages` array. What is the architectural flaw in this implementation of the agentic loop?

A) The `tool_result` blocks must be merged into a single JSON array within a `system` message, rather than using `user` messages.

B) The Anthropic API requires strictly alternating `user` and `assistant` messages; all three `tool_result` blocks must be placed inside a single `user` message.

C) The developer must append an `assistant` message acknowledging each `tool_result` between the `user` messages to maintain the conversation flow.

D) Multiple `tool_result` blocks cannot be processed in a single iteration; the developer must force Claude to request one tool at a time using `disable_parallel_tool_use`.


---

B is correct: The Anthropic Messages API enforces a strictly alternating pattern between 'user' and 'assistant' roles. When Claude generates multiple 'tool_use' blocks in a single assistant message, the corresponding 'tool_result' blocks must be returned collectively within the next single 'user' message to maintain this pattern.


## 35. Subdomain 1.5: Apply Agent SDK hooks for tool call interception and data normalization

A global logistics agent uses three different MCP tools to query delivery statuses from regional carriers. Tool A returns timestamps in Unix epoch, Tool B uses ISO 8601, and Tool C uses DD-MM-YYYY format. The model occasionally miscalculates delivery SLAs because it misinterprets the heterogeneous date formats. As an architect, how should you resolve this issue to guarantee accurate SLA calculations?

A) Implement a PostToolUse hook that detects the tool source, parses the specific date format into a standard ISO 8601 string, and replaces the raw output before returning the result to the model.

B) Update the system prompt with few-shot examples that show the model how to convert Unix epoch and DD-MM-YYYY timestamps into ISO 8601 before performing SLA calculations, ensuring consistent date handling across all carrier tools.

C) Create a separate DateConverter tool that the model calls via the system prompt whenever it receives a date from the logistics tools, converting Unix epoch and DD-MM-YYYY values into ISO 8601 before SLA calculations.

D) Implement a PreToolCall hook that injects a requested_format="ISO8601" parameter into the outgoing tool arguments for all three carrier tools, so that every response arrives in a uniform format for SLA computation.


---

A is correct: Implementing a PostToolUse hook is the architectural best practice for data normalization in agentic workflows. By intercepting the tool output and standardizing the date formats before the data reaches the model, you ensure that the model always receives consistent, high-quality data. This centralizes the logic and decouples data parsing from the LLM's reasoning.


## 36. Subdomain 1.7: Manage session state, resumption, and forking

An AI architect is currently managing two active sessions: Session A, which contains a large volume of raw system logs, and Session B, which contains the historical context and ongoing reasoning for a root cause analysis. The architect now needs to continue the root cause investigation and also begin a separate architecture analysis for a proposed fix. Which session management strategy is most appropriate?

A) Resume Session B; Start a new session for the architecture analysis.

B) Resume both Session A and Session B, as Claude can automatically detect stale log data.

C) Resume Session A; Start a new session for the log queries with a summary of the incident.

D) Start new sessions for both tasks, as any interruption requires a fresh context window.


---

A is correct as: This approach aligns with session management best practices. Resuming Session B preserves the valuable reasoning and narrative history of the investigation (stable context). Starting a new session for the architecture analysis ensures a clean, focused context window for the new task, isolating it from the investigation's history and preventing token waste.


## 37. Subdomain 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns

An HR assistant bot handles employee queries using a Coordinator and several subagents (PolicyAnalyzer, BenefitsCalculator, GeneralQA). For a simple query like "What are the cafeteria hours?", the bot takes 45 seconds to respond because the Coordinator routes the query through all subagents in a fixed pipeline before synthesizing the answer. Which design change will most effectively reduce latency for simple queries?

A) Redesign the Coordinator to evaluate query complexity upfront and dynamically route simple queries directly to the GeneralQA subagent, bypassing the others.

B) Broadcast the query to all subagents simultaneously and configure the Coordinator to immediately output the first response it receives.

C) Program the PolicyAnalyzer and BenefitsCalculator to return empty strings faster if they determine the query is irrelevant to their domain.

D) Deploy the subagents on faster, dedicated hardware to reduce the execution time of the fixed pipeline.


---

A is correct: Evaluating query complexity and intent upfront allows the Coordinator to implement dynamic routing. By identifying simple queries and routing them directly to the GeneralQA subagent, the system bypasses the overhead of the specialized subagents entirely. This 'fast path' eliminates unnecessary sequential processing, significantly reducing latency and compute costs.

B is wrong: Broadcasting queries to all subagents in parallel can reduce wall-clock time, but taking the 'first response' is dangerous in an HR context. It introduces non-determinism and risks providing inaccurate information if a specialized agent provides a generic but incomplete answer faster than the appropriate agent. It also wastes significant computational resources and does not solve the root problem of the fixed pipeline.