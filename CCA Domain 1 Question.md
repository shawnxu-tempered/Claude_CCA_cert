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