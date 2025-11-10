# Build A2A and MCP Systems using SWE Agents and agent-framework

## Lab Scenario

In this lab, you'll build a multi-agent event planning system using **[Microsoft Agent Framework](https://github.com/microsoft/agent-framework)**, an enterprise-ready framework that combines the best of Semantic Kernel and AutoGen. The framework's key differentiator is its **workflow orchestration capabilities**, which enable you to define complex multi-agent interactions as declarative workflows with clear execution paths, state management, and observability.

You'll create specialized AI agents that collaborate to plan comprehensive events, learning how to:

- **Build Multi-Agent Workflows**: Orchestrate multiple specialized agents (Event Coordinator, Venue Specialist, Budget Analyst, Catering Coordinator, and Logistics Manager) that work together through workflow edges and message passing. The framework supports **Agent-to-Agent (A2A) communication** for direct inter-agent messaging and **Model Context Protocol (MCP)** for integrating external tools like weather forecasting and calendar management.

- **Deploy to Azure AI Foundry**: Run your agent-framework workflows in Azure AI Foundry, where agents are automatically registered and managed with full enterprise-grade capabilities, security, and observability.

- **Visualize with DevUI and Azure AI Foundry**: Monitor real-time agent interactions, workflow execution graphs, and message flows through interactive interfaces both locally (DevUI) and in the cloud (Azure AI Foundry).

The system demonstrates concurrent workflow execution patterns where agents work in sequence and in parallel, exchange information through the workflow, invoke MCP tools for specialized capabilities, and synthesize comprehensive event plans. You'll also implement **human-in-the-loop** capabilities, allowing user input and approval at critical decision points during agent execution.

!IMAGE[Event Planning Agent Design.png](instructions310255/Event Planning Agent Design.png){700}

---

## Understanding Key Protocols: A2A and MCP

Before diving into the lab, let's understand two fundamental protocols that enable communication and collaboration in modern multi-agent systems:

### **Model Context Protocol (MCP)**

MCP is an open standard that defines how AI models access external tools and data sources. Think of it as a universal adapter:

- **Tool Discovery**: AI models can discover what tools are available and how to use them
- **Standardized Invocation**: Tools are called using a consistent interface, regardless of implementation
- **Bidirectional Integration**: Tools can be called by agents, and tools can invoke agents

**In this lab**, your agents use MCP tools like:
- **sequential-thinking-tools**: Advanced reasoning for complex planning tasks
- **Weather forecasting**: External API integration via MCP
- **Calendar management**: Persistent scheduling capabilities

### **Agent-to-Agent (A2A) Communication**

A2A is an emerging standard for enabling direct communication between AI agents. With A2A:

- **Direct Communication**: Agents can message each other directly, reducing latency and enabling more natural collaboration
- **Standardized Messaging**: A common protocol ensures agents built with different frameworks can communicate
- **Framework Compatibility**: Microsoft Agent Framework supports A2A for inter-agent communication patterns

Your workflow in this lab uses message passing between agents through workflow edges. Agent Framework also supports A2A communication patterns for scenarios requiring direct agent-to-agent communication beyond workflow orchestration.

---


===


## 1.  Lab Set-Up & Sign-In to Skillable Events GitHub

Sign in to your lab environment using:
- Username: +++@lab.VirtualMachine(Win11-Pro-Base).Username+++
- Password: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++

Your lab environment comes pre-configured with:

- **Visual Studio Code** - Primary development environment
- **Python 3.14** - Latest stable version
- **uv** - Fast Python package manager
- **Azure CLI** - Manage Azure resources
- **Azure Developer CLI (azd)** - Simplified deployment workflows
- **AI Toolkit extension for VSCode** - Simplifies Gen AI App Development
- **Git** - Source control

> [!hint] **Ignore Sign-In Notifications**
> 
> You may see blue "Sign in required" notifications or "Activate Windows" watermarks. Simply click "Not now" and continue - these won't affect your lab experience.

**Configure GitHub Copilot using *Skillable Events* GitHub Enterprise:**
1. From your Desktop (or Taskbar), open the **Edge** shortcut, and navigate to: +++**https://github.com/skillable-events**+++
1. Single sign-on to Skillable events, by clicking **Continue** on this page.

   !IMAGE[0-skillable-signin.png](instructions310255/0-skillable-signin.png)

1. Sign in with your Azure Cloud Credentials:
   - Username: 
   +++@lab.CloudPortalCredential(User1).Username+++
   
   - Temporary Access Pass: 
     +++@lab.CloudPortalCredential(User1).AccessToken+++

1. Click on Yes to Stay Signed-in, when prompted

---

## 2.  Sign In to GitHub Copilot

Before cloning the repository, you'll need to sign in to GitHub Copilot with the lab-provided GitHub Enterprise account.

> [!important] **Complete Step 1 First**
> 
> Ensure you've signed in to Skillable Events GitHub from Step 1 before proceeding.

1. Launch **Visual Studio Code** from the Taskbar.

2. In the bottom-right corner of VS Code, click the **GitHub Copilot** status icon, then click **Continue with GitHub**.

   !IMAGE[1-vsc-ghcp-signin.png](instructions310255/1-vsc-ghcp-signin.png)
   
   !IMAGE[1b-vsc-continue-w-github.png](instructions310255/1b-vsc-continue-w-github.png)

3. A browser window will open asking you to authorize Visual Studio Code. Click **Continue** to proceed.

4. On the authorization page, click **Authorize Visual-Studio-Code** to grant access.

   !IMAGE[2-authorize-vsc.png](instructions310255/2-authorize-vsc.png)

5. After authorization, the browser will redirect. Close the browser tab and return to VS Code.

6. Verify that the GitHub Copilot icon in the bottom-right now shows you're signed in.

> [!note] **GitHub Enterprise Account**
> 
> This lab uses a GitHub Enterprise account from the **skillable-events** organization, which provides access to GitHub Copilot features needed for this lab to work with specs.

---

===


## 3.  Clone and Set Up the Repository

1. In VS Code, open a new **Terminal** (*Terminal → New Terminal*).

2. Clone the Microsoft spec-to-agents repository and open the project:

   ```powershell
   # Clone the repository
   git clone https://github.com/microsoft/spec-to-agents.git
   cd spec-to-agents

   # TODO: Checkout Ignite lab branch

   # Open in VS Code
   code . --reuse-window

   ```

3. VS Code will reload with the **spec-to-agents** folder open.
4. When prompted with the workspace trust dialog, click **Yes, I trust the authors** to enable all features.

---

## 4  Start Azure Resource Provisioning (Background Process)

You'll use Azure Developer CLI (azd) to provision all necessary Azure resources. This process takes 5-10 minutes and runs in the background while you work on coding exercises.

1. **Authenticate with Azure CLIs** (ensure you're in the *spec-to-agents* root directory):

   First, authenticate with **azd**:
   
   **+++azd auth login+++**
   
   Select the existing signed-in account (from Step 1), or authenticate with:
   - Username: **+++@lab.CloudPortalCredential(User1).Username+++**  
   - Temporary Access Pass: **+++@lab.CloudPortalCredential(User1).AccessToken+++**
   
   Authenticate in your browser, then close the tab and return to VS Code.

   Next, authenticate with **Azure CLI**:
   
   **+++az login --use-device-code+++**
   
   Follow the device code instructions: copy the device code, click the link, select the existing account, and click **Continue** when prompted:
   
   !IMAGE[3-az-cli-continue.png](instructions310255/3-az-cli-continue.png)

   Return to VSCode and click enter to select the default subscription

2. **Create azd Environment**:

   **+++azd env new agents-lab-@lab.LabInstance.Id --subscription @lab.CloudSubscription.Id --location @lab.CloudResourceGroup(ResourceGroup1).Location+++**

3. **Start Provisioning** (do not wait for completion):
   
   **+++azd provision+++**

> [!knowledge] **Resources Being Provisioned**
> 
> This creates the following Azure resources:
> 
> **AI Platform:**
> - **Azure AI Foundry Project** - Project workspace for your event planning agents
> - **Model Deployments** - Two Azure OpenAI models deployed:
>   - *gpt-5-mini* (primary agent tasks)
>   - *gpt-4.1-mini* (web search capabilities)
> - **Bing Grounding Connection** - Real-time web search for agents
> 
> **Application Infrastructure:**
> - **Azure Container Apps Environment** - Managed serverless container hosting
> - **Azure Container App** - Hosts your multi-agent application with DevUI
> - **Azure Container Registry** - Stores and manages container images
> 
> **Monitoring & Security:**
> - **Application Insights** - Application monitoring and telemetry
> - **Log Analytics Workspace** - Centralized logging and analytics
> - **Managed Identity** - Secure authentication between services

**Proceed to coding exercises while provisioning runs in the background.**

---

===

## 5. Understanding the Codebase Structure

Before diving into coding, let's understand how the agent-framework code is organized.

**Open the `src/spec_to_agents/` package from repository root in VS Code Explorer** and examine this structure:
```
src/spec_to_agents/
├── agents/                    # Agent definitions (one per specialist)
│   ├── venue_specialist.py
│   ├── budget_analyst.py
│   ├── catering_coordinator.py
│   ├── logistics_manager.py
│   └── event_coordinator.py
├── prompts/                   # System prompts (agent instructions)
│   ├── venue_specialist.py
│   ├── budget_analyst.py
│   └── [... other agent files]
├── tools/                     # Tools and capabilities
│   ├── bing_search.py        # Custom web search @ai_function
│   ├── weather.py            # Weather forecast tool
│   ├── calendar.py           # Calendar management tools
│   └── mcp_tools.py          # MCP tool initialization
├── models/                    # Data models for structured outputs
│   └── messages.py           # SpecialistOutput, HumanFeedbackRequest
├── workflow/                  # Workflow orchestration logic
│   ├── core.py               # 🎯 Main workflow builder
│   └── executors.py          # EventPlanningCoordinator logic
├── utils/                     # Utility modules
│   ├── clients.py            # Azure AI client creation
│   ├── display.py            # Rich console formatting
│   └── constants.py          # Shared constants
├── config.py                  # Default model configuration
├── container.py               # Dependency injection container
├── main.py                    # 🚀 DevUI entry point
└── console.py                 # 💻 CLI entry point for testing
```

> [!knowledge] **Agent Framework Code Organization**
> 
> Agent Framework follows a clear separation of concerns:
> - **`/agents/`**: Agent creation functions that combine prompts + tools
> - **`/prompts/`**: System instructions defining agent behavior and expertise
> - **`/tools/`**: Reusable capabilities (web search, weather, calendars, MCP)
> - **`/workflow/`**: Orchestration logic connecting agents with edges
> - **`/models/`**: Type-safe data structures for agent communication
> - **`/utils/`**: Helper functions for client creation and display
> - **`main.py`**: Launch DevUI web interface (port 8080)
> - **`console.py`**: Run workflow in terminal with Rich formatting

---

===

## 6. Exercise 1: Create Your First Agent

**Concept**: In agent-framework, an **agent** is created by combining three elements:
1. **System instructions** (the "personality" and expertise)
2. **Tools** (capabilities like web search or code execution)
3. **Response format** (structured output for predictable responses)

> [!important] **Testing Comes Later**
> 
> You'll implement agent logic now, but won't run the workflow until **Exercise 12**. This allows `azd provision` (from Section 4) to complete in the background while you learn the framework through hands-on coding.

### Instructions

1. **Open** `src/spec_to_agents/agents/venue_specialist.py`

2. **Locate** the `# TODO: Exercise 1` comment (around line 42)

3. **Delete the `pass` statement**

4. **Uncomment and complete the agent creation code** by replacing the TODO section with:

```python
# TODO: Exercise 1 - Create Venue Specialist agent with web search capability

    # Initialize agent-specific tools
    agent_tools: list[ToolProtocol] = [web_search]

    if global_tools.get("sequential-thinking"):
        # Include MCP sequential-thinking tool from global tools
        agent_tools.append(global_tools["sequential-thinking"])

    return client.create_agent(
        name="venue_specialist",
        description="Expert in venue selection, site visits, and facility management for events.",
        instructions=venue_specialist.SYSTEM_PROMPT,
        tools=agent_tools,
        response_format=SpecialistOutput,
        **model_config,
    )
```

> [!note] **Don't worry about `web_search` errors yet!**
> 
> You'll see a red squiggle under `web_search` - that's expected! You'll implement this tool in Exercise 2. For now, just understand that it gives the agent web search capabilities.

> [!knowledge] **What You Just Learned**
> 
> **Agent Creation Pattern**:
> - `@inject` decorator: Enables automatic dependency injection - the framework provides `client`, `global_tools`, and `model_config` automatically
> - `client.create_agent()`: Factory method that creates all agents
> - `name`: Identifies the agent in workflow routing (must match IDs in `workflow/executors.py`)
> - `description`: Human-readable explanation shown in DevUI
> - `instructions`: System prompt from `prompts/venue_specialist.py` defining expertise
> - `tools`: List of capabilities - starts with `web_search`, conditionally adds MCP tool
> - `response_format`: Enforces structured JSON output using `SpecialistOutput` model
> - `**model_config`: Spreads configuration including `store=True` for service-managed threads
> 
> **Key Architecture Decisions**:
> 
> 1. **Dependency Injection**: No manual argument passing - dependencies "injected" by `container.py`. Cleaner code, better testability.
> 
> 2. **Service-Managed Threads**: Azure maintains conversation history automatically via `store=True` (in `model_config`). You don't track messages manually.
> 
> 3. **Dynamic Tool Loading**: Code checks if MCP tool exists (`global_tools.get("sequential-thinking")`) before adding. Makes agent work in different environments.
> 
> 4. **Structured Output**: `SpecialistOutput` ensures predictable JSON with `summary`, `next_agent`, `user_input_needed` fields for reliable workflow routing.

### Understanding Agent Tools

Your venue specialist starts with **one required tool**:
- `web_search`: Custom `@ai_function` for Bing-powered web search (Exercise 2)

Then **conditionally adds** an optional tool:
- `sequential-thinking`: MCP tool for advanced reasoning (if available)

This pattern (required tool + optional enhancement) is used across all specialist agents in the workflow.

---

===

## 7. Exercise 2: Implement a Web Search Tool

**Concept**: Tools in agent-framework are Python functions decorated with `@ai_function`. The LLM discovers and invokes these tools automatically when needed.

!IMAGE[Agent Tools Final.png](instructions310255/Agent Tools Final.png){700}

> [!important] **Testing Still Comes Later**
> 
> Continue building components - you'll test everything together in Exercise 12 after `azd provision` completes.

### Instructions

1. **Open** `src/spec_to_agents/tools/bing_search.py`

2. **Locate** the `# TODO: Exercise 2` comment (around line 36)

3. **Delete the `pass` statement**

4. **Uncomment and complete the tool implementation**:
```python
# TODO: Exercise 2 - Implement web search tool using HostedWebSearchTool

    # Ensure conflicting environment variables are not set
    os.environ.pop("BING_CUSTOM_CONNECTION_NAME", None)
    os.environ.pop("BING_CUSTOM_INSTANCE_NAME", None)
    try:
        web_search_tool = HostedWebSearchTool(description="Search the web for current information using Bing")

        # Use async context manager for proper cleanup
        async with create_agent_client() as client:
            agent = client.create_agent(
                name="bing_web_search_agent",
                tools=[web_search_tool],
                system_message=(
                    "You are a web search agent that uses the Bing Web Search tool to find information on the web."
                ),
                tool_choice=ToolMode.REQUIRED(function_name="web_search"),
                store=True,
                model_id=os.getenv("WEB_SEARCH_MODEL", "gpt-4.1-mini"),
            )
            response = await agent.run(f"Perform a web search for: {query}")
            return response.text
        # Agent automatically cleaned up when context manager exits

    except Exception as e:
        # Handle API errors gracefully
        error_type = type(e).__name__
        return f"Error performing web search: {error_type} - {e!s}"
```

> [!knowledge] **What You Just Learned**
> 
> **@ai_function Pattern**:
> - `@ai_function`: Decorator that auto-generates OpenAI function calling schemas from Python type hints
> - `Annotated[str, Field(description="...")]`: Provides parameter descriptions the LLM reads to understand when/how to call the tool
> - **Async functions**: All tools must be async (`async def`) for non-blocking I/O operations
> - **Error handling**: Always return `str` results, even for errors - LLMs can interpret error messages and adapt
> 
> **Tool-Agent Pattern (Agent-in-Tool)**:
> This implementation uses a **temporary agent** to invoke `HostedWebSearchTool`:
> 
> 1. **Create HostedWebSearchTool**: Built-in Azure AI tool that grounds responses with Bing Search
> 2. **Wrap in temporary agent**: Creates a focused agent just for this search
> 3. **Force tool use**: `tool_choice=ToolMode.REQUIRED` ensures the agent MUST call web_search
> 4. **Auto-cleanup**: `async with create_agent_client()` ensures resources are released
> 5. **Return text**: Extract just the text response for the calling agent
> 
> **Why use an agent-in-tool?**
> - HostedWebSearchTool requires an agent runtime to execute
> - Keeps search logic isolated and reusable
> - Automatic credential handling via Azure authentication
> - Built-in error handling and result formatting
> 
> **Environment Variable Cleanup**:
> The code removes `BING_CUSTOM_CONNECTION_NAME` and `BING_CUSTOM_INSTANCE_NAME` to ensure the tool uses the default Bing connection configured in Azure AI Foundry, preventing conflicts with custom search configurations.
> 
> **When is this called?**: 
> When the Venue Specialist (or any agent with this tool) needs venue information, the LLM will automatically invoke:
> ```python
> web_search("event venues in Seattle for 50 people under $3000")
> ```
> The LLM decides when to call based on the user's request and the tool's description.

### Understanding the Data Flow
```
User Request: "Find venues in Seattle"
    ↓
Venue Specialist Agent (detects need for web search)
    ↓
Calls: web_search("event venues Seattle")
    ↓
Creates temporary bing_web_search_agent
    ↓
Invokes: HostedWebSearchTool (Azure Bing API)
    ↓
Returns: Formatted search results
    ↓
Venue Specialist analyzes results and responds to user
```

This layered approach separates concerns: your agent handles reasoning, the tool handles execution, and the temporary agent handles Bing API interaction.

===

## 8. Exercise 3: Add MCP Sequential Thinking Tool

**Concept**: **MCP (Model Context Protocol)** is an open standard for connecting AI models to external tools. The `sequential-thinking` MCP server provides advanced reasoning capabilities to help agents break down complex planning tasks.

### Instructions

1. **Open** `src/spec_to_agents/tools/mcp_tools.py`

2. **Locate** the `# TODO: Exercise 3` comment (around line 47)

3. **Delete the `pass` statement**

4. **Uncomment and complete the MCP tool creation**:
```python
# TODO: Exercise 3 - Create MCP sequential thinking tool instance

    # Create MCP tool instance (not connected)
    sequential_thinking_tool = MCPStdioTool(
        name="sequential-thinking", 
        command="npx", 
        args=["-y", "@modelcontextprotocol/server-sequential-thinking"]
    )

    # Return tools dict for injection (framework manages lifecycle)
    return {"sequential-thinking": sequential_thinking_tool}
```

> [!knowledge] **What You Just Learned**
> 
> **MCP (Model Context Protocol)**:
> - Open standard for AI model ↔ tool communication (think "USB for AI tools")
> - Enables language models to discover and use external capabilities
> - MCP servers run as separate processes that agents communicate with
> - Standardized protocol means tools work across different AI frameworks
> 
> **MCPStdioTool**:
> - Connects to MCP servers via stdio (standard input/output)
> - `name`: Identifier used to reference the tool (e.g., `global_tools.get("sequential-thinking")`)
> - `command`: Process to launch (`npx` = Node.js package runner)
> - `args`: Command arguments - `["-y", "@modelcontextprotocol/server-sequential-thinking"]` means:
>   - `-y`: Auto-confirm package installation
>   - `@modelcontextprotocol/server-sequential-thinking`: Official MCP sequential thinking server
> 
> **Sequential Thinking Capabilities**:
> Helps agents with complex multi-step reasoning:
> - **Budget Analyst**: "Compare 3 allocation strategies: venue-focused vs balanced vs experience-focused"
> - **Venue Specialist**: "Evaluate venue options across capacity, location, cost, and amenities"
> - **Logistics Manager**: "Plan timeline considering setup, event flow, vendor coordination, and breakdown"
> 
> **Framework-Managed Lifecycle**:
> Notice there's NO `async with` context manager here - that's intentional!
> - **Tool Creation** (this function): Creates tool instance but does NOT connect to MCP server
> - **Lazy Connection**: Framework connects automatically on first use via `__aenter__()`
> - **Automatic Cleanup**: Framework handles disconnection via:
>   - DevUI mode: `_cleanup_entities()` calls `tool.close()`
>   - Console mode: `ChatAgent` exit stack manages lifecycle
> 
> You never manually start/stop the MCP server - the framework handles it!
> 
> **Dictionary Return Pattern**:
> Returning `dict[str, ToolProtocol]` allows:
> - Multiple MCP tools in one place (future: add more tools like "memory-tool", "filesystem-tool")
> - Easy lookup by name: `global_tools.get("sequential-thinking")`
> - Injection via DI container as `global_tools` parameter
> 
> **MCP Tool Types**:
> There are three MCP tool types depending on communication protocol:
> - **MCPStdioTool**: Uses stdio (standard input/output) - what we're using
> - **MCPStreamableHTTPTool**: Uses HTTP/SSE for streaming responses
> - **MCPWebsocketTool**: Uses WebSockets for bidirectional communication

### Understanding the Architecture
```
Dependency Injection Container (container.py)
    ↓
global_tools = Singleton(create_mcp_tool_instances)
    ↓
Returns: {"sequential-thinking": MCPStdioTool(...)}
    ↓
Injected into: All agent create_agent() functions
    ↓
if global_tools.get("sequential-thinking"):
    agent_tools.append(global_tools["sequential-thinking"])
    ↓
Framework connects MCP server on first agent.run()
```

**Key Insight**: This pattern (create → inject → lazy connect) means:
- No overhead if agents don't use the tool
- Automatic cleanup prevents resource leaks
- Shared tool instance across all agents (efficient)

---

===

## 9. Exercise 4: Define Structured Output Format

**Concept**: **Structured outputs** ensure agents return predictable, parseable responses instead of free-form text. This enables reliable workflow routing and eliminates brittle text parsing.

### Instructions

1. **Open** `src/spec_to_agents/models/messages.py`

2. **Locate** the `# TODO: Exercise 4` comment (around line 66)

3. **Delete the `pass` statement**

4. **Uncomment and complete the Pydantic model**:
```python
# TODO: Exercise 4 - Define structured output fields for agent responses

    summary: str = Field(description="Concise summary of this specialist's recommendations (max 200 words)")
    next_agent: Literal["venue", "budget", "catering", "logistics"] | None = Field(
        description=(
            "ID of next agent to route to ('venue', 'budget', 'catering', 'logistics'), or None if done/need user input, requests to other sub-agents from a sub-agent should always go through the coordinator (e.g. venue → event_coordinator → budget)"
        )
    )
    user_input_needed: bool = Field(default=False, description="Whether user input is required before proceeding")
    user_prompt: str | None = Field(default=None, description="Question to ask user if user_input_needed=True")
```

> [!knowledge] **What You Just Learned**
> 
> **Structured Outputs with Pydantic**:
> - `BaseModel`: Pydantic base class providing type validation, serialization, and schema generation
> - `Field(description=...)`: Provides semantic hints to the LLM about each field's purpose and valid values
> - **Type hints with validation**: 
>   - `Literal["venue", "budget", "catering", "logistics"]`: Restricts `next_agent` to only valid agent IDs
>   - `str | None`: Means "string or null" - LLM understands optional fields
>   - `bool`: Boolean field for yes/no decisions
> - **Default values**: `default=False` provides fallback if field is omitted in LLM response
> 
> **Why Structured Outputs Matter**:
> 
> **Before structured outputs** (free-form text):
> ```python
> # Agent returns: "I think we should check with the budget analyst next, 
> # but first let me ask the user which venue they prefer..."
> 
> # Your code needs fragile parsing:
> if "budget" in response.lower():
>     next_agent = "budget"
> if "ask" in response or "user" in response:
>     user_input_needed = True
> # Breaks easily! What if text mentions "budget" in a different context?
> ```
> 
> **With structured outputs** (Pydantic model):
> ```python
> # Agent MUST return valid JSON:
> {
>   "summary": "Found 3 venues...",
>   "next_agent": "budget",
>   "user_input_needed": false,
>   "user_prompt": null
> }
> 
> # Your code is simple and reliable:
> if output.user_input_needed:
>     await ctx.request_info(...)
> elif output.next_agent:
>     await ctx.send_message(..., target_id=output.next_agent)
> else:
>     await synthesize_final_plan()
> ```
> 
> **Using Literal for Type Safety**:
> The `Literal["venue", "budget", "catering", "logistics"]` type means:
> - **Validation**: Only these exact strings are valid - typos like `"budjet"` cause errors
> - **LLM guidance**: The LLM sees the allowed values in the schema and won't hallucinate invalid IDs
> - **IDE autocomplete**: Your code editor suggests valid values when accessing `output.next_agent`
> 
> **How It Works in the Workflow**:
> ```python
> # In agent creation (agents/venue_specialist.py):
> agent = client.create_agent(
>     name="venue_specialist",
>     response_format=SpecialistOutput,  # Forces structured output
>     ...
> )
> 
> # Agent's response is automatically validated:
> response = await agent.run(user_message)
> output = response.value  # Already parsed as SpecialistOutput!
> 
> # Coordinator uses fields for routing (workflow/executors.py):
> if output.user_input_needed:
>     # Pause for human input
> elif output.next_agent:
>     # Route to specified agent
> else:
>     # Workflow complete
> ```

### Example Responses

**Autonomous routing** (agent proceeds to next specialist):
```json
{
  "summary": "Researched 3 venues: The Foundry ($2k, 60 capacity, modern downtown), Pioneer Square Hall ($1.5k, 50 capacity, historic), Fremont Studios ($3k, 80 capacity, industrial). Recommended The Foundry for best balance of location, capacity, and budget.",
  "next_agent": "budget",
  "user_input_needed": false,
  "user_prompt": null
}
```

**Request user input** (agent needs clarification):
```json
{
  "summary": "Found 3 excellent venue options with different tradeoffs in price, location, and ambiance. Each meets your capacity requirement of 50 people.",
  "next_agent": null,
  "user_input_needed": true,
  "user_prompt": "Which venue style appeals to you: The Foundry (modern, $2k), Pioneer Square Hall (historic, $1.5k), or Fremont Studios (industrial, $3k)?"
}
```

**Workflow complete** (final specialist finished):
```json
{
  "summary": "Created comprehensive event timeline: setup 2-5pm, doors 6pm, reception 6:30pm, dinner 7pm, program 8pm, close 10pm. Weather forecast clear, all vendor coordination confirmed, calendar events created.",
  "next_agent": null,
  "user_input_needed": false,
  "user_prompt": null
}
```

### Understanding the Routing Logic

The coordinator uses this pattern (in `workflow/executors.py`):
```python
specialist_output = parse_specialist_output(response)

if specialist_output.user_input_needed:
    # Pause workflow and request human input via DevUI
    await ctx.request_info(...)
    
elif specialist_output.next_agent:
    # Route to specified specialist
    await ctx.send_message(..., target_id=specialist_output.next_agent)
    
else:
    # Both fields are None/False → workflow complete
    await synthesize_final_plan()
```

This clean routing logic is only possible because of structured outputs - no text parsing or regex needed!

---

===

## 10. Exercise 5: Build the Workflow with Edges

**Concept**: The **WorkflowBuilder** defines how agents communicate through **edges**. Each edge represents a potential message-passing path between executors.

### Instructions

1. **Open** `spec_to_agents/workflow/core.py`

2. **Locate** the `# TODO: Exercise 5` comment (around line 89)

3. **Delete the `pass` statement**

4. **Uncomment and complete the workflow builder**:
```python
# TODO: Exercise 5 - Build the event planning workflow with agent executors

    # Create agents
    coordinator_agent = event_coordinator.create_agent()
    venue_agent = venue_specialist.create_agent()
    budget_agent = budget_analyst.create_agent()
    catering_agent = catering_coordinator.create_agent()
    logistics_agent = logistics_manager.create_agent()
    # Create coordinator executor with routing logic
    coordinator = EventPlanningCoordinator(coordinator_agent)

    # Create specialist executors
    venue_exec = AgentExecutor(agent=venue_agent, id="venue")
    budget_exec = AgentExecutor(agent=budget_agent, id="budget")
    catering_exec = AgentExecutor(agent=catering_agent, id="catering")
    logistics_exec = AgentExecutor(agent=logistics_agent, id="logistics")

    # Build workflow with bidirectional star topology
    workflow = (
        WorkflowBuilder(
            name="Event Planning Workflow",
            description=(
                "Multi-agent event planning workflow with venue selection, budgeting, "
                "catering, and logistics coordination. Supports human-in-the-loop for "
                "clarification and approval."
            ),
            max_iterations=30,  # Prevent infinite loops
        )
        # Set coordinator as start executor
        .set_start_executor(coordinator)
        # Bidirectional edges: Coordinator ←→ Each Specialist
        .add_edge(coordinator, venue_exec)
        .add_edge(venue_exec, coordinator)
        .add_edge(coordinator, budget_exec)
        .add_edge(budget_exec, coordinator)
        .add_edge(coordinator, catering_exec)
        .add_edge(catering_exec, coordinator)
        .add_edge(coordinator, logistics_exec)
        .add_edge(logistics_exec, coordinator)
        .build()
    )

    # Set stable ID to prevent URL issues on restart
    workflow.id = "event-planning-workflow"
    return workflow
```

> [!knowledge] **What You Just Learned**
> 
> **Workflow Builder Pattern**:
> - **`WorkflowBuilder`**: Fluent API for constructing workflows with method chaining
> - **`.set_start_executor()`**: Defines workflow entry point (receives initial user message)
> - **`.add_edge(from_executor, to_executor)`**: Adds directed communication path between executors
> - **`.build()`**: Validates graph structure and creates immutable workflow instance
> - **`max_iterations`**: Safety limit prevents infinite loops in cyclic workflows
> 
> **Dependency Injection in Action**:
> Notice how agent creation is simple:
> ```python
> coordinator_agent = event_coordinator.create_agent()  # No arguments!
> venue_agent = venue_specialist.create_agent()         # DI provides dependencies
> ```
> 
> The `@inject` decorator on `build_event_planning_workflow` and on each `create_agent()` function enables automatic parameter injection. The DI container (from `container.py`) provides:
> - `client`: Azure AI agent client
> - `global_tools`: MCP tools dictionary
> - `model_config`: Agent configuration
> 
> **Executor Types**:
> - **`EventPlanningCoordinator`**: Custom executor (extends `Executor` class) with:
>   - `@handler` methods for routing logic
>   - `@response_handler` for human-in-the-loop
>   - Access to `WorkflowContext` for `ctx.send_message()` and `ctx.request_info()`
> 
> - **`AgentExecutor`**: Standard wrapper for ChatAgent instances
>   - Simple pass-through: receives message → runs agent → returns response
>   - `id` parameter crucial for routing: `SpecialistOutput.next_agent="budget"` routes to executor with `id="budget"`
> 
> **Star Topology Architecture**:
> ```
> Linear (rigid):
>     Venue → Budget → Catering → Logistics
>     (agents execute in fixed order)
> 
> Star (flexible):
>              Venue
>                ↑↓
>     Budget ← Coordinator → Catering
>                ↑↓
>             Logistics
>     (coordinator routes dynamically based on SpecialistOutput.next_agent)
> ```
> 
> **Why Bidirectional Edges?**
> Each specialist needs two edges:
> 1. **Coordinator → Specialist**: Send work requests
> 2. **Specialist → Coordinator**: Return `SpecialistOutput` for routing decisions
> 
> Without bidirectional edges, specialists couldn't report back to the coordinator!
> 
> **Workflow Execution Flow**:
> ```python
> # User starts workflow
> result = await workflow.run("Plan a corporate party for 50 people")
> 
> # Execution trace:
> 1. User message → Coordinator (start_executor)
> 2. Coordinator reads request → routes to venue specialist
>    await ctx.send_message(..., target_id="venue")
> 
> 3. Venue specialist searches venues → returns:
>    SpecialistOutput(
>        summary="Found 3 venues...",
>        next_agent="budget",
>        user_input_needed=False
>    )
> 
> 4. Coordinator receives response → routes to budget:
>    await ctx.send_message(..., target_id="budget")
> 
> 5. Budget analyst allocates costs → returns:
>    SpecialistOutput(next_agent="catering", ...)
> 
> 6. Coordinator routes to catering → then logistics → then synthesizes final plan
> 
> 7. Coordinator yields final output:
>    await ctx.yield_output(final_plan)
> ```

### Understanding the Graph Structure

The workflow creates this execution graph:

!IMAGE[Event Planning Agent Design.png](instructions310255/Event Planning Agent Design.png){700}

**Key Benefits**:
- **Dynamic routing**: Coordinator decides next agent based on `SpecialistOutput.next_agent`
- **Human-in-the-loop**: Any specialist can request user input, coordinator handles it
- **Parallel potential**: Star topology could support parallel execution (future enhancement)
- **Flexible flow**: Order isn't hardcoded - venue could route to catering, skipping budget if needed

### Stable Workflow ID
```python
workflow.id = "event-planning-workflow"
```

This sets a stable identifier for the workflow, which:
- Creates consistent DevUI URLs: `http://localhost:8080/workflow/event-planning-workflow`
- Prevents URL changes on restart (random IDs would break bookmarks)
- Enables workflow versioning and tracking in production

---
===


## 11. Verify Environment Setup

The `azd provision` command (from Section 4) automatically configured everything. Let's verify it completed.

### Instructions

1. **Check `azd provision` finished successfully**

   Look for this message in your terminal from Section 4:
   ```
   SUCCESS: Deployment complete!
   ```

   > [!note] **Still Running?**
   > 
   > Wait for `azd provision` to complete before proceeding (typically 5-10 minutes).

2. **Verify `.env` file exists**

   ```powershell
   Test-Path .env
   ```

   Should output: `True`

> [!knowledge] **What Happened**
> 
> The post-provisioning hook automatically:
> - Generated `.env` with Azure configuration
> - Installed dependencies via `uv sync`
> - Created `.venv` virtual environment

> [!note] **If `.env` is missing**, run:
> ```powershell
> .\scripts\generate_env.ps1
> ```

---

===

## 12. Run Your First Multi-Agent Workflow

Now let's see your agents in action!

### Instructions

1. **Start the console application**:
   ```powershell
   uv run console
   ```

2. **Wait for the welcome screen** with agent descriptions:
   
   ```
   ╭─────────────────────────────────────────────────────────────────────╮
   │                      Event Planning Workflow                        │
   │              Interactive CLI with AI-Powered Agents                 │
   ╰─────────────────────────────────────────────────────────────────────╯

   This workflow uses specialized AI agents to help you plan events:
     • Venue Specialist - Researches and recommends venues
     • Budget Analyst - Manages costs and financial planning
     • Catering Coordinator - Handles food and beverage
     • Logistics Manager - Coordinates schedules and resources

   ✓ Workflow loaded successfully
   ```

3. **Enter your event request**:
   
   ```
   Your request (or 1-3 for examples): Host an Ignite after party for 100 in SF downtown near Moscone Center on Nov 20th 2025 with a budget of $10k
   ```

4. **Watch the workflow execute**. You'll see each agent work sequentially:

   ```
   ──────────────────────── venue ────────────────────────
   {"summary":"Found The Mezzanine near Moscone Center, capacity 120, 
   $2.5k rental...","next_agent":"budget",...}

   ──────────────────────── budget ────────────────────────
   {"summary":"Budget allocation: Venue $2.5k, Catering $5k, AV $1.5k, 
   Contingency $1k...","next_agent":"catering",...}

   ──────────────────────── catering ────────────────────────
   {"summary":"Cocktail reception with passed appetizers and open bar, 
   $50/person...","next_agent":"logistics",...}

   ──────────────────────── logistics ────────────────────────
   [Tool calls: create_calendar_event, get_weather_forecast]
   {"summary":"Timeline: Setup 5pm, doors 7pm, event 7-10pm. Weather 
   forecast: clear, 55°F...","next_agent":null,...}
   ```

5. **Review the final event plan**:
   
   ```
   ──────────────────── Final Event Plan ────────────────────

   ╭──────────────────── Event Plan Summary ─────────────────╮
   │                                                          │
   │  Executive Summary                                       │
   │  Ignite after party for 100 guests at The Mezzanine,    │
   │  SF downtown near Moscone Center, Nov 20th 2025.        │
   │                                                          │
   │  Venue: The Mezzanine ($2.5k)                           │
   │  Budget: $10k total allocated                           │
   │  Catering: Cocktail reception with open bar             │
   │  Logistics: 7-10pm, weather clear, calendar confirmed   │
   │                                                          │
   ╰──────────────────────────────────────────────────────────╯

   ✓ Event planning complete!
   ```

> [!knowledge] **What Just Happened?**
> 
> **Workflow Execution**:
> 1. **Venue Specialist** → Searched SF venues near Moscone, selected The Mezzanine, routed to `budget`
> 2. **Budget Analyst** → Allocated $10k budget, routed to `catering`
> 3. **Catering Coordinator** → Designed cocktail menu, routed to `logistics`
> 4. **Logistics Manager** → Created timeline, checked weather, confirmed calendar, returned `next_agent=null`
> 5. **Event Coordinator** → Synthesized final plan
> 
> **Key Features**:
> - **Structured routing**: Each agent's `next_agent` field drove the workflow path
> - **Autonomous tool use**: Agents decided when to call tools (web search, calendar, weather)
> - **Service-managed threads**: Each agent automatically remembered prior context
> - **No human input needed**: Workflow ran autonomously with complete constraints

---
===

## 13. BONUS: Visualize with DevUI

Run the workflow with a visual interface for debugging and exploration.

1. **Start DevUI** (from repository root):
   ```powershell
   uv run app
   ```
   Browser opens at `http://localhost:8080`

2. **Run the Workflow**:
   - Select **"Event Planning Workflow"** from the top dropdown
   - Click **"Configure & Run"**
   - Enter your prompt: `Plan a corporate holiday party for 50 people, budget $5000`
   - Click **"Run Workflow"**

   !IMAGE[devui-run-workflow.png](instructions310255/devui-run-workflow.png){700}

3. **Watch Execution**:
   - **Graph**: Nodes light up green as agents execute
   - **Events Tab**: Real-time agent outputs and structured JSON responses
   - **Traces Tab**: Execution duration and routing decisions
   - **Tools Tab**: Tool calls (like `web_search`) with arguments and results

   !IMAGE[devui-execution.png](instructions310255/devui-execution.png){900}

4. **Run the Agents**
   - Select **"VenueSpecialist"** from the top dropdown
   - Click the text box at the bottom of the page
   - Enter your prompt: `Plan a corporate holiday party for 50 people, budget $5000`
   - Click the **Tools** tab on the right hand side of the screen
   - Inspect the call to the **web_search** tool

> [!knowledge] **DevUI vs Console**
> 
> - **Visual workflow graph**: See agent communication patterns
> - **Detailed traces**: Debug with execution timings and message routing
> - **Tool inspection**: View all tool calls and results in one place


---
===
## 14. BONUS: Explore Supervisor Pattern (Alternative Branch)

The main branch uses a **star topology** where a coordinator routes between specialists. There's an alternative **supervisor pattern** available on a different branch.

!IMAGE[Event Planning Agent Design Bonus.png](instructions310255/Event Planning Agent Design Bonus.png){800}

1. **Switch to the supervisor branch**:
   ```powershell
   git fetch origin
   git checkout feature/main-with-supervisor-workflow
   ```

2. **Run the supervisor workflow**:
   ```powershell
   uv run console
   ```

3. **Compare the patterns**:

   **Star Topology (main branch)**:
   ```
   User → Coordinator → Venue → Coordinator → Budget → Coordinator → ...
   ```
   - Coordinator reads `SpecialistOutput.next_agent` to route
   - Specialists don't directly communicate
   - Coordinator synthesizes final output

   **Supervisor Pattern (bonus branch)**:
   ```
   User → Supervisor
            ├─→ Venue ──┐
            ├─→ Budget ─┤
            ├─→ Catering┤→ Supervisor → Final Plan
            └─→ Logistics┘
   ```
   - Supervisor delegates tasks in parallel
   - Specialists work independently
   - Supervisor aggregates all results at once

4. **When to use each**:
   - **Star**: Sequential dependencies (budget depends on venue)
   - **Supervisor**: Independent tasks (can run in parallel)

---
===

## 15. Understanding What You Built

Let's recap the key agent-framework concepts you've learned:

### **Core Abstractions**

| Concept | What It Is | Where You Used It |
|---------|-----------|------------------|
| **Agent** | ChatAgent with instructions + tools | `venue_specialist.create_agent()` |
| **Tool** | `@ai_function` that agents can invoke | `web_search()`, MCP sequential thinking |
| **Executor** | Wrapper for workflow integration | `AgentExecutor(agent, id="venue")` |
| **Workflow** | Graph of agents connected by edges | `WorkflowBuilder().add_edge(...)` |
| **Structured Output** | Pydantic model for predictable responses | `SpecialistOutput` |

### **Key Patterns**

**1. Agent Creation**:
```python
agent = client.create_agent(
    name="AgentName",
    instructions=SYSTEM_PROMPT,
    tools=[tool1, tool2],
    response_format=OutputModel,
    store=True  # Automatic conversation history
)
```

**2. Tool Definition**:
```python
@ai_function
async def my_tool(
    param: Annotated[str, Field(description="Help text for LLM")]
) -> str:
    # Implementation
    return result
```

**3. Workflow Building**:
```python
workflow = (
    WorkflowBuilder(name="My Workflow")
    .set_start_executor(start)
    .add_edge(from_executor, to_executor)
    .build()
)
```

**4. Structured Output**:
```python
class MyOutput(BaseModel):
    summary: str
    next_agent: str | None
```

### **Why Agent Framework?**

- **Enterprise-Ready**: Built on proven Semantic Kernel infrastructure
- **Multi-Agent Orchestration**: Native support for complex workflows
- **Azure Integration**: Works seamlessly with Azure AI Foundry
- **Observability**: Built-in tracing and monitoring
- **Type Safety**: Pydantic models and Python type hints
- **Flexible**: Supports both orchestrated and autonomous patterns

---
===

## 16. Deploy to Azure

Now let's deploy your workflow to Azure!

1. **Ensure `azd provision` completed** (from Section 4). Check the terminal for success message.

2. **Deploy the application**:
   ```powershell
   cd ..  # Back to repository root
   azd deploy
   ```

3. **Wait for deployment** (3-5 minutes). You'll see:
   ```
   Packaging services...
   Building container image...
   Pushing to Azure Container Registry...
   Deploying to Azure Container Apps...
   
   SUCCESS: Deployed to https://devui-....azurecontainerapps.io
   ```

4. **Note the DevUI URL** from the output.

---
===

## 17. Explore Azure AI Foundry

Your agents are also running in Azure AI Foundry!

1. **Open Azure AI Foundry**: Navigate to +++**https://ai.azure.com**+++

2. **Sign in** with your Azure credentials (from Section 1)

3. **Open your project**:
   - Click **All resources**
   - Select **agents-lab-@lab.LabInstance.Id**

4. **View Deployed Agents**:
   - Left navigation → **Build** → **Agents**
   - You'll see all 5 specialist agents listed
   - Click any agent to view:
     - System instructions
     - Configured tools
     - Model deployment

   !IMAGE[ai-foundry-agents.png](instructions310255/ai-foundry-agents.png)

5. **Test in Playground**:
   - Click **Playground** tab
   - Select **Event Planning Workflow**
   - Enter a prompt: "Plan a product launch event for 100 people"
   - Watch the workflow execute in the cloud

6. **View Execution Traces**:
   - Click **Tracing** in left navigation
   - Select a recent workflow run
   - Explore:
     - Agent transitions
     - Tool invocations
     - Token usage
     - Latency metrics

7. **Monitor with Application Insights**:
   - Navigate to **Azure Portal** → Your Resource Group
   - Click **Application Insights** resource
   - View:
     - **Live Metrics**: Real-time performance
     - **Application Map**: Service dependencies
     - **Performance**: Request duration and failures

---
===

## 18. Clean-Up

> [!alert] **Important: Remove Azure Resources**
> 
> To avoid charges, delete all Azure resources created in this lab.

1. **Delete Azure resources**:
   ```powershell
   azd down --purge --force
   ```

2. **Confirm deletion** when prompted. This removes:
   - Azure AI Foundry project
   - Model deployments
   - Container Apps
   - Container Registry
   - All associated resources

3. **Sign out** from services (Optional - lab will be purged automatically):
   - VS Code: Click profile icon → **Sign Out**
   - Edge: Sign out from GitHub, Azure Portal, AI Foundry

---
===

## Congratulations!

You've successfully built and deployed a production-ready multi-agent system! Here's what you mastered:

### **✅ Core Skills Acquired**

- **Agent Framework Fundamentals**:
  - Created specialized agents with tools and instructions
  - Implemented `@ai_function` tools for custom capabilities
  - Used structured outputs for predictable agent responses
  - Built multi-agent workflows with WorkflowBuilder

- **Protocol Integration**:
  - **A2A**: Agent-to-agent message passing through edges
  - **MCP**: Integrated external tools via Model Context Protocol

- **Production Deployment**:
  - Deployed to Azure AI Foundry
  - Monitored with Application Insights
  - Visualized with DevUI

### **🎯 Key Takeaways**

1. **Agents are composable**: Instructions + Tools + Response Format
2. **Tools enable capabilities**: Web search, code execution, MCP servers
3. **Workflows orchestrate**: Edges define communication paths
4. **Structured outputs**: Enable reliable routing and parsing
5. **Azure integration**: Enterprise-scale deployment with monitoring

### **🚀 Next Steps**

Continue your agent-framework journey:

- **Add more agents**: Create a Marketing Specialist or Transportation Coordinator
- **Custom MCP tools**: Build domain-specific tool servers
- **Advanced workflows**: Implement conditional branching and loops
- **Production features**: Add authentication, rate limiting, cost tracking

### **📚 Resources**

- **Agent Framework**: [github.com/microsoft/agent-framework](https://github.com/microsoft/agent-framework)
- **Azure AI Foundry**: [ai.azure.com](https://ai.azure.com)
- **MCP Protocol**: [modelcontextprotocol.io](https://modelcontextprotocol.io)

**Happy coding at Microsoft Ignite 2025!**