# Build A2A and MCP Systems using SWE Agents and agent-framework

## Lab Scenario

In this lab, you'll build a multi-agent event planning system using **[Microsoft Agent Framework](https://github.com/microsoft/agent-framework)**, an enterprise-ready framework that combines the best of Semantic Kernel and AutoGen. The framework's key differentiator is its **workflow orchestration capabilities**, which enable you to define complex multi-agent interactions as declarative workflows with clear execution paths, state management, and observability.

You'll create specialized AI agents that collaborate to plan comprehensive events, learning how to:

- **Build Multi-Agent Workflows**: Orchestrate multiple specialized agents (Event Coordinator, Venue Specialist, Budget Analyst, Catering Coordinator, and Logistics Manager) that work together through workflow edges and message passing. The framework supports **Agent-to-Agent (A2A) communication** for direct inter-agent messaging and **Model Context Protocol (MCP)** for integrating external tools like weather forecasting and calendar management.

- **Deploy to Microsoft Foundry**: Run your agent-framework workflows in Microsoft Foundry, where agents are automatically registered and managed with full enterprise-grade capabilities, security, and observability.

- **Visualize with DevUI and Microsoft Foundry**: Monitor real-time agent interactions, workflow execution graphs, and message flows through interactive interfaces both locally (DevUI) and in the cloud (Microsoft Foundry).

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
- **Python 3.12** - Latest stable version
- **uv** - Fast Python package manager
- **Azure CLI** - Manage Azure resources
- **Azure Developer CLI (azd)** - Simplified deployment workflows
- **AI Toolkit extension for VSCode** - Simplifies Gen AI App Development
- **Microsoft Foundry extension for VSCode** - Unified platform for Enterprise AI operations, Model builders, and App development
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

   - If Temp pass didn't work, try
     +++@lab.CloudPortalCredential(User1).Password+++

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

# Checkout Ignite lab branch
git checkout ignite-2025

# Open in VS Code
code . --reuse-window

   ```

3. VS Code will reload with the **spec-to-agents** folder open.
4. When prompted with the workspace trust dialog, click **Yes, I trust the authors** to enable all features.

### Install dependencies:

> ⚠️ **In VS Code**: Open the integrated terminal (*Ctrl+* or `Terminal > New Terminal`)

```powershell
# Install dependencies (enables IntelliSense and prepares local environment)
uv sync
```

> [!hint] **Save Time: Start Azure Authentication in Parallel**
> 
> While `uv sync` is running, open a **new terminal** (*Terminal → New Terminal* or click the **+** icon) and proceed to Section 4 to authenticate with Azure. This lets both processes run simultaneously!

---
## 4  Start Azure Resource Provisioning (Background Process)
You'll use Azure Developer CLI (azd) to provision all necessary Azure resources. This process takes 5-10 minutes and runs in the background while you work on coding exercises.

> [!note] **Parallel Execution**
> 
> If `uv sync` is still running from Section 3, that's fine! Open a new terminal for these commands so both can run in parallel.

1. **Open a new terminal** in VS Code (*Terminal → New Terminal* or click the **+** icon)

2. **Authenticate with Azure CLIs** (ensure you're in the *spec-to-agents* root directory):
   
   First, authenticate with **azd**:
   
   **+++azd auth login+++**
   
   Select the existing signed-in account (from Step 1), or authenticate with:
   - Username: **+++@lab.CloudPortalCredential(User1).Username+++**  
   - Temporary Access Pass: **+++@lab.CloudPortalCredential(User1).AccessToken+++**
   - If Temp pass didn't work, try **+++@lab.CloudPortalCredential(User1).Password+++**
   
   Authenticate in your browser, then close the tab and return to VS Code.
   
   Next, authenticate with **Azure CLI**:
   
   **+++az login+++**
   
   A sign-in dialog will appear. Click **Work or school account**, then click **Continue**:
   
   !IMAGE[azlogin-work-school.png](instructions310255/azlogin-work-school.png)
   

> [!note]  You may need to minimize VS Code to see the sign-in window.   

   
   Sign in with your Azure credentials:
   - Username: **+++@lab.CloudPortalCredential(User1).Username+++**  
   - Temporary Access Pass: **+++@lab.CloudPortalCredential(User1).AccessToken+++**
   - If Temp pass didn't work, try **+++@lab.CloudPortalCredential(User1).Password+++**
   
   !IMAGE[azlogin-signinwithazure.png](instructions310255/azlogin-signinwithazure.png)
   
   When prompted about device registration, click **No, this app only**:
   
   !IMAGE[azcli-thisdevice.png](instructions310255/azcli-thisdevice.png)
   
   Return to VS Code terminal and press **Enter** to select the default subscription.
   ---

===

## 4  Start Azure Resource Provisioning (Background Process) - continued.

2. **Create azd Environment**:
   
   **+++azd env new agents-lab-@lab.LabInstance.Id --subscription @lab.CloudSubscription.Id --location @lab.CloudResourceGroup(ResourceGroup1).Location+++**

3. **Start Provisioning** (do not wait for completion):
   
   **+++azd provision+++**

> [!knowledge] **Resources Being Provisioned**
> 
> This creates the following Azure resources:
> 
> **AI Platform:**
> - **Microsoft Foundry Project** - Project workspace for your event planning agents
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

2. **Locate** the `# TODO: Exercise 1` comment (around line 48)

3. **Delete the `pass` statement**

4. **Uncomment and complete the agent creation code** by replacing the TODO section with:

```python
# Exercise 1 - Create Venue Specialist agent with web search capability

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

> [!note] **Don't worry about `web_search` yet!**
> 
> You'll implement this tool in Exercise 2. For now, just understand that it gives the agent web search capabilities.

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
> 
> ### Understanding Agent Tools
> 
> Your venue specialist starts with **one required tool**:
> - `web_search`: Custom `@ai_function` for Bing-powered web search (Exercise 2)
> 
> Then **conditionally adds** an optional tool:
> - `sequential-thinking`: MCP tool for advanced reasoning (if available)
> 
> This pattern (required tool + optional enhancement) is used across all specialist agents in the workflow.

---

===

## 7. Exercise 2: Implement a Web Search Tool

**Concept**: Tools in agent-framework are Python functions decorated with `@ai_function`. The LLM discovers and invokes these tools automatically when needed.

!IMAGE[Agent Tools.png](instructions310255/Agent Tools.png)

> [!important] **Testing Still Comes Later**
> 
> Continue building components - you'll test everything together in Exercise 12 after `azd provision` completes.

### Instructions

1. **Open** `src/spec_to_agents/tools/bing_search.py`

2. **Locate** the `# TODO: Exercise 2` comment (around line 40)

3. **Delete the `pass` statement**

4. **Uncomment and complete the tool implementation**:
```python
# Exercise 2 - Implement web search tool using HostedWebSearchTool

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
> The code removes `BING_CUSTOM_CONNECTION_NAME` and `BING_CUSTOM_INSTANCE_NAME` to ensure the tool uses the default Bing connection configured in Microsoft Foundry, preventing conflicts with custom search configurations.
> 
> **When is this called?**: 
> When the Venue Specialist (or any agent with this tool) needs venue information, the LLM will automatically invoke:
> ```python
> web_search("event venues in Seattle for 50 people under $3000")
> ```
> The LLM decides when to call based on the user's request and the tool's description.
> 
> ### Understanding the Data Flow
> ```
> User Request: "Find venues in Seattle"
>     ↓
> Venue Specialist Agent (detects need for web search)
>     ↓
> Calls: web_search("event venues Seattle")
>     ↓
> Creates temporary bing_web_search_agent
>     ↓
> Invokes: HostedWebSearchTool (Azure Bing API)
>     ↓
> Returns: Formatted search results
>     ↓
> Venue Specialist analyzes results and responds to user
> ```
> 
> This layered approach separates concerns: your agent handles reasoning, the tool handles execution, and the temporary agent handles Bing API interaction.

===

## 8. Exercise 3: Add MCP Sequential Thinking Tool

**Concept**: **MCP (Model Context Protocol)** is an open standard for connecting AI models to external tools. The `sequential-thinking` MCP server provides advanced reasoning capabilities to help agents break down complex planning tasks.

### Instructions

1. **Open** `src/spec_to_agents/tools/mcp_tools.py`

2. **Locate** the `# TODO: Exercise 3` comment (around line 43)

3. **Delete the `pass` statement**

4. **Uncomment and complete the MCP tool creation**:
```python
# Exercise 3 - Create MCP sequential thinking tool instance

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
> 
> ### Understanding the Architecture
> ```
> Dependency Injection Container (container.py)
>     ↓
> global_tools = Singleton(create_mcp_tool_instances)
>     ↓
> Returns: {"sequential-thinking": MCPStdioTool(...)}
>     ↓
> Injected into: All agent create_agent() functions
>     ↓
> if global_tools.get("sequential-thinking"):
>     agent_tools.append(global_tools["sequential-thinking"])
>     ↓
> Framework connects MCP server on first agent.run()
> ```
> 
> **Key Insight**: This pattern (create → inject → lazy connect) means:
> - No overhead if agents don't use the tool
> - Automatic cleanup prevents resource leaks
> - Shared tool instance across all agents (efficient)

---

===

## 9. Exercise 4: Define Structured Output Format

**Concept**: **Structured outputs** ensure agents return predictable, parseable responses instead of free-form text. This enables reliable workflow routing and eliminates brittle text parsing.

### Instructions

1. **Open** `src/spec_to_agents/models/messages.py`

2. **Locate** the `# TODO: Exercise 4` comment (around line 76)

3. **Delete the `pass` statement**

4. **Uncomment and complete the Pydantic model**:
```python
# Exercise 4 - Define structured output fields for agent responses

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
>
> ### Example Responses
> 
> **Autonomous routing** (agent proceeds to next specialist):
> ```json
> {
>   "summary": "Researched 3 venues: The Foundry ($2k, 60 capacity, modern downtown), Pioneer Square Hall ($1.5k, 50 capacity, historic), Fremont Studios ($3k, 80 capacity, industrial). Recommended The Foundry for best balance of location, capacity, and budget.",
>   "next_agent": "budget",
>   "user_input_needed": false,
>   "user_prompt": null
> }
> ```
> 
> **Request user input** (agent needs clarification):
> ```json
> {
>   "summary": "Found 3 excellent venue options with different tradeoffs in price, location, and ambiance. Each meets your capacity requirement of 50 people.",
>   "next_agent": null,
>   "user_input_needed": true,
>   "user_prompt": "Which venue style appeals to you: The Foundry (modern, $2k), Pioneer Square Hall (historic, $1.5k), or Fremont Studios (industrial, $3k)?"
> }
> }
> ```
> 
> **Workflow complete** (final specialist finished):
> ```json
> {
>   "summary": "Created comprehensive event timeline: setup 2-5pm, doors 6pm, reception 6:30pm, dinner 7pm, program 8pm, close 10pm. Weather forecast clear, all vendor coordination confirmed, calendar events created.",
>   "next_agent": null,
>   "user_input_needed": false,
>   "user_prompt": null
> }
> ```
> 
> ### Understanding the Routing Logic
> 
> The coordinator uses this pattern (in `workflow/executors.py`):
> ```python
> specialist_output = parse_specialist_output(response)
> 
> if specialist_output.user_input_needed:
>     # Pause workflow and request human input via DevUI
>     await ctx.request_info(...)
>     
> elif specialist_output.next_agent:
>     # Route to specified specialist
>     await ctx.send_message(..., target_id=specialist_output.next_agent)
>     
> else:
>     # Both fields are None/False → workflow complete
>     await synthesize_final_plan()
> ```
> 
> This clean routing logic is only possible because of structured outputs - no text parsing or regex needed!

---

===

## 10. Exercise 5: Build the Workflow with Edges

**Concept**: The **WorkflowBuilder** defines how agents communicate through **edges**. Each edge represents a potential message-passing path between executors.

### Instructions

1. **Open** `spec_to_agents/workflow/core.py`

2. **Locate** the `# TODO: Exercise 5` comment (around line 85)

3. **Delete the `pass` statement**

4. **Uncomment and complete the workflow builder**:
```python
# Exercise 5 - Build the event planning workflow with agent executors

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
> 
> ### Understanding the Graph Structure
> 
> The workflow creates this execution graph:
> 
> !IMAGE[Event Planning Agent Design.png](instructions310255/Event Planning Agent Design.png){700}
> 
> **Key Benefits**:
> - **Dynamic routing**: Coordinator decides next agent based on `SpecialistOutput.next_agent`
> - **Human-in-the-loop**: Any specialist can request user input, coordinator handles it
> - **Parallel potential**: Star topology could support parallel execution (future enhancement)
> - **Flexible flow**: Order isn't hardcoded - venue could route to catering, skipping budget if needed
> 
> ### Stable Workflow ID
> ```python
> workflow.id = "event-planning-workflow"
> ```
> 
> This sets a stable identifier for the workflow, which:
> - Creates consistent DevUI URLs: `http://localhost:8080/workflow/event-planning-workflow`
> - Prevents URL changes on restart (random IDs would break bookmarks)
> - Enables workflow versioning and tracking in production

---
===


## 11. Verify Environment Setup

The `azd provision` command (from Section 4) automatically configured everything. Let's verify it completed.

### Instructions

1. **Check `azd provision` finished successfully**

   Look for this message in your terminal from Section 4:
   ```
   SUCCESS: Your application was provisioned in Azure ...
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
> .\scripts\generate-env.ps1
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

2. **Wait for the welcome screen**:
   
   ```
   ╭─────────────────────────────────────────────────────────────────────╮
   │                      Event Planning Workflow                        │
   │              Interactive CLI with AI-Powered Agents                 │
   ╰─────────────────────────────────────────────────────────────────────╯

   ✓ Workflow loaded successfully
   ```

3. **Enter your event request**:
   
   ```
   Host an Ignite after party for 100 in SF downtown near Moscone Center on Nov 20th 2025 with a budget of $10k
   ```

4. **Watch the workflow execute**. Agents work collaboratively, sometimes iterating:

   ```
   ──────────────────────── venue ────────────────────────
   [Tool: web_search for venues near Moscone]
   {"summary":"Recommended LongHouse (83 Minna St), capacity 250, 
   built-in bar, walkable from Moscone...","next_agent":"budget",...}

   ──────────────────────── budget ────────────────────────
   {"summary":"Initial allocation: Venue $5k (50%), Catering $2.5k (25%), 
   AV $800 (8%)...","next_agent":"catering",...}

   ──────────────────────── catering ────────────────────────
   {"summary":"Quality catering needs $4k for passed appetizers + hosted bar. 
   Current $2.5k insufficient...","next_agent":"budget",...}

   ──────────────────────── budget ────────────────────────
   {"summary":"Revised: Venue $3.5k (35%), Catering $4k (40%), 
   AV $800, Rentals $700...","next_agent":"catering",...}

   ──────────────────────── catering ────────────────────────
   {"summary":"Plan fits $4k: Passed hors d'oeuvres, hot station, 
   hosted beer/wine first hour...","next_agent":"logistics",...}

   ──────────────────────── logistics ────────────────────────
   [Tool: create_calendar_event]
   [Tool: get_weather_forecast]
   {"summary":"Timeline: Load-in 5:30pm, doors 8pm, event 8-11pm. 
   Calendar confirmed...","next_agent":null,...}
   ```

5. **Review the final event plan** - synthesized from all agent outputs:
   
   ```
   ──────────────────── Final Event Plan ────────────────────

   ╭──────────────────── Event Plan Summary ─────────────────╮
   │  Ignite after party for 100 guests, Nov 20, 2025       │
   │  LongHouse (83 Minna St), near Moscone Center          │
   │                                                         │
   │  Budget: $10k (Venue $3.5k, Catering $4k, AV/Rentals)  │
   │  Menu: Passed appetizers, hot station, hosted bar      │
   │  Timeline: 8-11pm, calendar confirmed                  │
   ╰─────────────────────────────────────────────────────────╯

   ✓ Event planning complete!
   ```

> [!knowledge] **What Just Happened?**
> 
> **Dynamic Workflow**:
> - **Venue** searched web → recommended LongHouse
> - **Budget** allocated $10k → sent to catering
> - **Catering** found allocation insufficient → routed back to budget
> - **Budget** reallocated funds → sent revised plan to catering
> - **Catering** approved revised budget → sent to logistics
> - **Logistics** created timeline + calendar → completed workflow
> 
> **Key Observations**:
> - **Non-linear routing**: Agents can route back to previous agents when needed
> - **Autonomous negotiation**: Budget and catering iterated to find optimal allocation
> - **Tool invocation**: Venue used web search, logistics used calendar/weather
> - **Service-managed threads**: Each agent remembered full conversation context

---
===

## 13. Visualize with DevUI

Run the workflow with a visual interface for debugging and exploration.

1. **Start DevUI** (from repository root):
   ```powershell
uv run app
   ```
   Browser opens at `http://localhost:8080`

2. **Run the Workflow**:
   - Click **"Configure & Run"**
   - Enter your prompt: `Plan a corporate holiday party for 50 people, budget $5000`
   - Click **"Run Workflow"** and respond to any user inputs (elicitations)

   !IMAGE[devui-run-workflow.png](instructions310255/devui-run-workflow.png){700}
   !IMAGE[devui-1-user-elicitation.png](instructions310255/devui-1-user-elicitation.png){700}

3. **Watch Execution**:
   - **Graph**: Nodes light up green as agents execute
   - **Events Tab**: Real-time agent outputs and structured JSON responses
   - **Traces Tab**: Execution duration and routing decisions
   - **Tools Tab**: Tool calls (like `web_search`) with arguments and results

   !IMAGE[devui-3-finish-run.png](instructions310255/devui-3-finish-run.png){700}
   !IMAGE[tool-calls-devui.png](instructions310255/tool-calls-devui.png){300}

4. **Run the Agents** (Optional)
   - Select **"VenueSpecialist"** from the top dropdown
   - Click the text box at the bottom of the page
   - Enter your prompt: `Plan a corporate holiday party for 50 people, budget $5000`
   - Respond to any user elicitation requests
   - Click the **Tools** tab on the right hand side of the screen
   - Inspect the call to the **web_search** tool

   !IMAGE[devui-agent.png](instructions310255/devui-agent.png){600}

> [!knowledge] **DevUI vs Console**
> 
> - **Visual workflow graph**: See agent communication patterns
> - **Detailed traces**: Debug with execution timings and message routing
> - **Tool inspection**: View all tool calls and results in one place


---

===

## 14. Deploy to Azure

Now let's deploy your workflow to Azure!

1. **Ensure `azd provision` completed** (from Section 4). Check the terminal for success message.

2. **Deploy the application**:
   ```powershell
   azd deploy
   ```

3. **Wait for deployment** (3-5 minutes). You'll see output similar to:
   ```
   Deploying services (azd deploy)
     Publishing service app (Uploading remote build context)
     Publishing service app (Running remote build)
     (✓) Done: Deploying service app
     - Endpoint: https://ca-xxxxxxx.<region>.azurecontainerapps.io/
   ```

4. **Open the deployed DevUI**:
   - Copy the Endpoint URL from the deployment output
   - Open it in your browser
   - Test the workflow with the same prompt you used locally: `Plan a corporate holiday party for 50 people, budget $5000`
   - Verify that the deployed version works identically to your local DevUI

---

===

## 15. Explore Observability in Microsoft Foundry

See how Microsoft Foundry provides enterprise observability for your multi-agent system.

### Access Microsoft Foundry

1. **Navigate to** +++**https://ai.azure.com**+++ and click **Sign in**

2. **Authenticate** with your credentials (or use existing signed-in account):
   - Username: +++**@lab.CloudPortalCredential(User1).Username**+++
   - Temporary Access Pass: +++**@lab.CloudPortalCredential(User1).AccessToken**+++
   - If Temp pass didn't work, try +++**@lab.CloudPortalCredential(User1).Password**+++

3. **Select your project** (there will be only one):

   !IMAGE[aif-select-project.png](instructions310255/aif-select-project.png){300}

---

### View Distributed Tracing

1. **Navigate to** **Tracing** under "Observe and optimize" in the left navigation

2. **Click "Connect"** to connect to Application Insights (only one option available)

!IMAGE[connect-appinsights.png](instructions310255/connect-appinsights.png)

3. **View workflow runs** - each **workflow.run** entry shows a complete event planning execution:

   !IMAGE[tracing-workflow-run.png](instructions310255/tracing-workflow-run.png){300}

4. **Click any workflow.run** to see the distributed trace:
   - Hierarchical view of all agents (venue, budget, catering, logistics)
   - Tool calls (web_search, sequentialthinking)
   - Expand nodes to see timing, token usage, and input/output data

   !IMAGE[tracing-explore.png](instructions310255/tracing-explore.png){500}

> [!knowledge] **Distributed Tracing Benefits**
> 
> - Track requests across all agents and tools
> - Identify performance bottlenecks
> - Debug failures with complete execution context
> - Monitor token usage and costs

---

### Inspect Agent Configurations

1. **Navigate to** **Agents** under "Build" in the left navigation

2. **Click "event_coordinator"** to view:
   - System instructions matching your local code
   - Configured tools (web_search, sequentialthinking)
   - Model deployment (gpt-5-mini)

   !IMAGE[agents-event-coordinator.png](instructions310255/agents-event-coordinator.png){300}

3. **Explore Threads** to see conversation history from your console runs

---

### Use VS Code Extension

1. **Click the Microsoft Foundry icon** in VS Code's left sidebar (bottom icon):

   !IMAGE[aif-vsc-extension.png](instructions310255/aif-vsc-extension.png){200}

2. **Sign in to Azure** (if prompted) using the same credentials

3. **Expand "RESOURCES"** to explore:
   - **Models**: Deployed models and settings
   - **Agents**: All agents with quick access
   - **Threads**: Recent conversation threads

> [!knowledge] **Observability Layers**
> 
> - **Tracing**: HOW the workflow executed (timing, routing, failures)
> - **Agents**: WHAT each component does (instructions, tools)
> - **Threads**: Conversation history and state

---
===
## 16. BONUS: A2A (Agent-to-Agent) Integration

In this bonus exercise, you'll explore **Agent-to-Agent (A2A) communication** by integrating your event planning workflow with external A2A-compatible agents. A2A enables direct agent communication across different frameworks and deployments.

### What is A2A?

A2A is an emerging standard protocol that enables:
- **Cross-Framework Communication**: Agents built with different frameworks can communicate
- **Distributed Agent Systems**: Agents can run in different environments and still collaborate
- **Standardized Messaging**: Common protocol for agent-to-agent requests and responses

### Exercise: Integrate an A2A Agent

1. **Open a new VS Code window** for the A2A samples. From your current terminal, run:
   ```powershell
code -n
   ```
   This opens a new VS Code instance in the same lab environment.

2. **In the new VS Code window**, open a terminal and clone the A2A samples repository:
   ```powershell
git clone https://github.com/a2aproject/a2a-samples.git
cd a2a-samples\samples\python\agents\azureaifoundry_sdk\azurefoundryagent
code . --reuse-window
   ```

3. Open a new Terminal and **Create a `.env` file** from the template:
   ```powershell
cp .env.template .env
   ```

4. **Configure the A2A agent** with values from your spec-to-agents `.env` file:
   
   Open the `.env` file you just created and update:
   ```
AZURE_AI_FOUNDRY_PROJECT_ENDPOINT=<Your Microsoft Foundry Project Endpoint From Spec-to-Agents>
AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME=gpt-5-mini
   ```
   
   > [!hint] **Finding Your Endpoint**
   > 
   > The `AZURE_AI_FOUNDRY_PROJECT_ENDPOINT` value is in your spec-to-agents `.env` file. You can find it by opening the `.env` file in your original VS Code window (the spec-to-agents project).

5. **Run the A2A agent**:
   ```powershell
uv run .
   ```
   
   The A2A agent will start and listen for requests from other agents. Keep this terminal running.

6. **Return to your original VS Code window** (spec-to-agents project) and open the `.env` file.

7. **Enable A2A integration** by uncommenting the `A2A_AGENT_HOST` variable:
   ```
A2A_AGENT_HOST=http://localhost:10007
   ```
   
   Save the file.

9. **Test the A2A integration via DevUI**:
   
   From your *spec-to-agents* repository, start DevUI (if not already running):
   ```powershell
uv run app
   ```
   
   DevUI will launch and open in your browser at `http://localhost:8080`

10. **Select the AI Foundry Calendar Agent**:
    - In DevUI, click the **Agents** dropdown at the top (currently shows "Workflows")
    - Select **"AI Foundry Calendar Agent"** from the list
    
    !IMAGE[devui-calendar-agent.png](instructions310255/devui-calendar-agent.png){400}

11. **Interact with the A2A agent**:
    - In the chat input at the bottom, type: `Check my availability for Friday at 8 am`
    - Press Enter and watch the response
    - Try other queries like:
      - `Schedule a meeting for tomorrow at 2 PM`
      - `What events do I have next week?`

12. **Observe the A2A communication**:
    - **Switch to your A2A VS Code window** and check the terminal where `uv run .` is running
    - You'll see console logs showing incoming requests from your spec-to-agents DevUI interaction
    - The logs show the A2A protocol in action - your agent-framework workflow communicating with the Azure AI Foundry Agent that is A2A compatible

13. **View the agent in Microsoft Foundry**:
    - Navigate to +++**https://ai.azure.com**+++
    - Sign in and select your project
    - Go to **Build** → **Agents** in the left navigation
    - You'll see **foundry-calendar-agent** listed alongside your event planning agents
    - Click on it to view:
      - The agent configuration
      - Conversation threads from your DevUI interactions
      - Tool calls and responses
    
    !IMAGE[Foundry-agents-calendar.png](instructions310255/Foundry-agents-calendar.png){400}

> [!knowledge] **What Just Happened?**
> 
> You've just demonstrated cross-framework agent communication:
> 
> **The Flow:**
> 1. **DevUI / Agent Framework** (spec-to-agents) → sent user request via A2A protocol to...
> 2. **A2A compatabile Azure AI Foundry Agent** (foundry-calendar-agent) → processed A2A request using Azure AI Foundry's agent capabilities
> 
> **Key Insight:** The Azure AI Foundry Agent is an instance of Azure's native agent service, but your agent-framework agents can communicate with it seamlessly through the A2A protocol. This demonstrates true interoperability across different agent frameworks and deployment models.

> [!knowledge] **A2A in Production**
> 
> In production scenarios, A2A enables:
> - **Specialized Agent Services**: Deploy domain-expert agents (like calendar management) as microservices
> - **Multi-Organization Collaboration**: Agents from different organizations can communicate securely
> - **Flexible Scaling**: Scale individual agent services independently based on demand
> - **Framework Agnostic**: Use the best tool for each job - Azure AI Foundry for some agents, custom frameworks for others

---
===

## 17. BONUS: Implement Entertainment Agent with Spec-Kit

In this bonus exercise, you'll use **spec-kit** - a specification-driven development toolkit - to implement a new Entertainment Agent using GitHub Copilot. Instead of coding from scratch, you'll use pre-defined specifications and let Copilot implement the feature.

### What is Spec-Kit?

Spec-kit flips traditional development: **specifications become executable**. You define WHAT you want (requirements), HOW to build it (plan), and WHAT steps to take (tasks), then let AI agents implement the code.

**Traditional Approach**:
```
Write code → Test → Debug → Refactor → Document
```

**Spec-Kit Approach**:
```
Constitution (principles) → Specify (what) → Plan (how) → Tasks (steps) → Implement (AI does it)
```

For this lab, we've **pre-populated** all specifications. You'll focus on the implementation workflow.

---

### Step 1: Review Pre-Populated Specifications

The `.specify/` directory contains all the specifications for the Entertainment Agent feature:

1. **Open** `.specify/memory/constitution.md`
   - Project principles and coding standards
   - These guide ALL implementation decisions

2. **Open** `.specify/specs/002-entertainment-agent/spec.md`
   - Feature requirements and user stories
   - Defines WHAT the Entertainment Agent does

3. **Open** `.specify/specs/002-entertainment-agent/plan.md`
   - Technical architecture and patterns
   - Defines HOW to implement the agent

4. **Open** `.specify/specs/002-entertainment-agent/tasks.md`
   - Sequential implementation steps
   - What Copilot will execute

> [!knowledge] **Key Insight**
> 
> Notice the separation of concerns:
> - **Constitution**: Principles (never changes)
> - **Spec**: Requirements (WHAT and WHY)
> - **Plan**: Architecture (HOW)
> - **Tasks**: Actions (EXECUTE)

---

### Step 2: Switch to Claude Sonnet 4.5

For optimal spec-kit performance, use the Claude Sonnet 4.5 model:

1. **Click** the GitHub Copilot icon in VS Code (bottom right)

2. **Select** model dropdown (currently shows your active model)

3. **Choose** `Claude Sonnet 4.5` from the list

---

### Step 3: Start Spec-Kit Implementation

Now you'll trigger Copilot to implement all tasks:

1. **Open GitHub Copilot Chat** (Ctrl+Shift+I or click Copilot icon)

2. **Type** the spec-kit command:
   ```
/speckit.implement implement Tasks
   ```
!IMAGE[speckit-impl.png](instructions310255/speckit-impl.png)

3. **Press Enter** - Copilot will start reading the tasks.md file

4. **Watch the initial analysis** - Copilot will:
   - Read tasks.md
   - Identify 14 tasks to complete
   - Show a summary of files to create/modify

!IMAGE[speckit-read-files.png](instructions310255/speckit-read-files.png){500}

---

### Step 4: Approve Copilot Requests

As Copilot implements tasks, it will request permission to modify files:

1. **Copilot shows proposed changes**:
   - File path and change preview
   - Type of change (create, modify, delete)

2. **Review the change** (quick scan is fine)

3. **Click "Accept"** or press **Ctrl+Enter**

4. **Repeat** for each task


> [!tip] **Approval Tips**
> 
> - **First approval** (Task 0.1): Updates `models/messages.py` - CRITICAL for routing
> - **Most approvals**: Creating new files - safe to accept
> - **If uncertain**: Check the file diff before approving

**Expected approvals** (14 total):
- Task 0.1: Modify `models/messages.py` ✅ CRITICAL FIRST
- Task 1.1: Create `prompts/entertainment_specialist.py`
- Task 1.2: Create `agents/entertainment_specialist.py`
- Task 1.3: Modify `workflow/executors.py`
- Task 1.4: Modify `workflow/builder.py`
- Task 2.1: Modify `prompts/logistics_manager.py`
- Task 2.2: Modify `prompts/event_coordinator.py`
- Task 3.1: Create `tests/functional/test_entertainment_agent.py`
- ... (Copilot may combine some changes)

---

### Step 5: Learn About Spec-Kit (While Copilot Works)

While Copilot implements the tasks (~5-10 minutes), let's understand what's happening:

#### **How Spec-Kit Works**

**1. Constitution** (`.specify/memory/constitution.md`)
- Defines project principles: "Use dependency injection", "Follow framework patterns"
- Copilot references these when making decisions
- Ensures consistency across all features

**2. Specification** (`.specify/specs/002-entertainment-agent/spec.md`)
- **User Stories**: What the feature should do
  - US-1: Research entertainment options
  - US-2: Coordinate with other agents
  - US-3: Output structured recommendations
- **Requirements**: Functional and non-functional constraints
- **Success Metrics**: How to know it works

**3. Plan** (`.specify/specs/002-entertainment-agent/plan.md`)
- **Architecture**: Which files to create/modify
- **Patterns**: Code examples to follow
- **Integration**: How to connect to existing workflow
- **Testing Strategy**: How to validate

**4. Tasks** (`.specify/specs/002-entertainment-agent/tasks.md`)
- **Sequential Steps**: Task 0.1 → 1.1 → 1.2 → ... → 3.4
- **Dependencies**: Which tasks must complete before others
- **Acceptance Criteria**: How to verify each task
- **File Paths**: Exact locations for implementation

#### **Why This Approach?**

Traditional coding:
```
Developer writes code → Trial and error → Fix bugs → Refactor → Document
```

Spec-kit approach:
```
Specifications written → AI implements → Tests validate → Production-ready code
```

**Benefits**:
- ✅ **Consistency**: All code follows same patterns
- ✅ **Speed**: AI implements faster than manual coding
- ✅ **Quality**: Guided by project principles
- ✅ **Documentation**: Specs serve as documentation
- ✅ **Onboarding**: New team members read specs to understand system

#### **What's Being Implemented?**

The **Entertainment Agent** will:
1. **Research entertainment options** using web search
2. **Respect budget allocation** from Budget Agent
3. **Validate venue capabilities** (stage, A/V equipment)
4. **Coordinate timing** with Logistics Agent
5. **Return structured recommendations** to Event Coordinator

This demonstrates the complete agent-framework pattern you learned in Exercises 1-12.

---

### Step 6: Verify Implementation Complete

After Copilot finishes (~10 minutes), verify all changes:

1. **Check Copilot's summary** - should show:
   ```
   ✅ Task 0.1: Updated SpecialistOutput model
   ✅ Task 1.1: Created system prompt
   ✅ Task 1.2: Created agent module
   ✅ Task 1.3: Added executor class
   ✅ Task 1.4: Integrated into workflow
   ✅ Task 2.1: Updated logistics routing
   ✅ Task 2.2: Updated coordinator synthesis
   ✅ Task 3.1: Created functional tests
   ```

2. **Verify files created**:
   ```powershell
   # Check new files exist
   Test-Path src/spec_to_agents/agents/entertainment_specialist.py
   Test-Path src/spec_to_agents/prompts/entertainment_specialist.py
   Test-Path tests/functional/test_entertainment_agent.py
   ```
   All should return `True`

3. **Check for errors** in Copilot output:
   - ✅ No red error messages
   - ✅ All tasks marked complete
   - ✅ No unresolved conflicts

!IMAGE[spec-kit-impl-done.png](instructions310255/spec-kit-impl-done.png){500}

---

### Step 7: Test in Console Mode

Now test the complete workflow with the new Entertainment Agent:

1. **Start console mode**:
   ```powershell
   uv run console
   ```

2. **Select example 1** (or press `1` and Enter):
   ```
   Plan a corporate holiday party for 50 people, budget $5000
   ```

3. **When prompted for location**, type:
   ```
   Seattle
   ```

4. **Watch the workflow execute** through all agents:
   ```
   ─────── venue ───────
   [Venue Specialist researches venues...]
   Recommended: The Foundry Seattle
   
   ─────── budget ───────
   [Budget Analyst allocates funds...]
   Allocation: Venue $2,500, Catering $1,400, Entertainment $500...
   
   ─────── catering ───────
   [Catering Coordinator plans menu...]
   
   ─────── logistics ───────
   [Logistics Manager creates timeline...]
   
   ─────── entertainment ───────  ← NEW AGENT!
   [Entertainment Specialist searches options...]
   Recommended: Local jazz band ($400), DJ ($350), Trivia host ($250)
   
   ─────── coordinator ───────
   [Event Coordinator synthesizes final plan...]
   ```

5. **Verify Entertainment section in final plan**:
   - Entertainment recommendations with costs
   - Timing coordination with event schedule
   - Budget compliance ($500 allocated)

!IMAGE[speckit-enter-1.png](instructions310255/speckit-enter-1.png)

!IMAGE[speckit-enter-2.png](instructions310255/speckit-enter-2.png)

> [!knowledge] **What Just Happened?**
> 
> The workflow now follows this path:
> ```
> User → Coordinator → Venue → Budget → Catering → Logistics → Entertainment → Coordinator
> ```
> 
> The Entertainment Agent:
> 1. Received context from previous agents (budget, venue, timing)
> 2. Used web_search tool to find options
> 3. Returned structured output with `next_agent=None`
> 4. Coordinator synthesized all results including entertainment

---

### Step 8: Test in DevUI

Finally, verify the Entertainment Agent appears in the visual workflow:

1. **Start DevUI** (in a new terminal):
   ```powershell
   # In a new PowerShell terminal
   cd C:\Users\LabUser\code\spec-to-agents
   uv run app
   ```

2. **Open browser** to the DevUI URL (shown in terminal):
   ```
   http://localhost:8080
   ```

3. **Submit the same test prompt**:
   ```
   Plan a corporate holiday party for 50 people in Seattle, budget $5000
   ```

4. **Watch the workflow visualization**:
   - **Graph View**: Entertainment Specialist node appears
   - **Execution Trace**: Shows entertainment processing step
   - **Output Panel**: Displays entertainment recommendations

5. **Explore the Entertainment Agent**:
   - Click the **Entertainment Specialist** node in graph
   - View agent details:
     - System instructions
     - Available tools (web_search, sequential-thinking)
     - Response format (SpecialistOutput)

!IMAGE[speckit-enter-devui.png](instructions310255/speckit-enter-devui.png)

---

### Step 9: Reflection - What Did Spec-Kit Enable?

Let's discuss what just happened:

#### **Compare: Traditional vs Spec-Kit**

**If you coded this manually**:
- ⏱️ 60-90 minutes to implement
- 🐛 Trial and error with routing logic
- 📝 Manual pattern matching from existing agents
- 🧪 Writing tests from scratch
- 🔍 Debugging integration issues

**With spec-kit**:
- ⏱️ 10 minutes to review specs + approve changes
- ✅ Patterns guaranteed to match existing code
- ✅ Tests automatically generated
- ✅ Integration tested via structured outputs
- ✅ Documentation created alongside code

#### **Key Learnings**

1. **Specifications Drive Development**
   - Constitution defines HOW to code (principles)
   - Spec defines WHAT to build (requirements)
   - Plan defines WHERE and HOW (architecture)
   - Tasks define WHEN (sequence)

2. **AI Implements Patterns**
   - Copilot followed existing agent patterns exactly
   - Dependency injection used correctly
   - Structured outputs enforced
   - Tests matched existing test structure

3. **Type Safety Catches Errors**
   - Task 0.1 updated `SpecialistOutput` model first
   - Without it, runtime would fail with validation error
   - Pydantic ensures correct routing

4. **Workflow Extension is Repeatable**
   - Want to add Risk Management Agent? Follow same process
   - Want to add Marketing Agent? Same specifications approach
   - Patterns are documented and reproducible

#### **Discussion Questions**

- How would you add a sixth agent (e.g., Marketing Specialist)?
- What would Task 0.1 need to include for a new agent?
- When would you use spec-kit vs manual coding?
- How does constitution.md ensure code quality?



---

===
## 18. Understanding What You Built

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

### **Advanced Patterns (Bonus Section 17)**

**5. Spec-Driven Development**:
```
Constitution → Specification → Plan → Tasks → /speckit.implement
```

**6. Type-Safe Routing**:
```python
next_agent: Literal["venue", "budget", "catering", "logistics", "entertainment"] | None
```

### **Why Agent Framework?**

- **Enterprise-Ready**: Built on proven Semantic Kernel infrastructure
- **Multi-Agent Orchestration**: Native support for complex workflows
- **Azure Integration**: Works seamlessly with Microsoft Foundry
- **Observability**: Built-in tracing and monitoring
- **Type Safety**: Pydantic models and Python type hints
- **Flexible**: Supports both orchestrated and autonomous patterns
- **Extensible**: Spec-kit enables rapid feature development

---
===

## 19. Explore Microsoft Foundry

Your agents are also running in Microsoft Foundry!

1. **Open Microsoft Foundry**: Navigate to +++**https://ai.azure.com**+++

2. **Sign in** with your Azure credentials (from Section 1)

3. **Open your project**:
   - Click **All resources**
   - Select **agents-lab-@lab.LabInstance.Id**

4. **View Deployed Agents**:
   - Left navigation → **Build** → **Agents**
   - You'll see all 5 specialist agents listed (or 6 if you completed Section 17!)
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

## 20. Clean-Up

Delete all Azure resources created in this lab (Optional - lab will be purged automatically).

1. **Delete Azure resources**:
   ```powershell
   azd down --purge --force
   ```

2. **Confirm deletion** when prompted. This removes:
   - Microsoft Foundry project
   - Model deployments
   - Container Apps
   - Container Registry
   - All associated resources

3. **Sign out** from services:
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
  - Deployed to Microsoft Foundry
  - Monitored with Application Insights
  - Visualized with DevUI

- **Bonus: A2A Integration** (Section 16):
  - Integrated external A2A-compatible agents
  - Explored cross-framework agent communication
  - Implemented distributed agent collaboration patterns

- **Bonus: Spec-Driven Development** (Section 17):
  - Used spec-kit to implement Entertainment Agent
  - Followed specification-driven workflow
  - Extended multi-agent system with new capability
  - Applied type-safe routing patterns

### **🎯 Key Takeaways**

1. **Agents are composable**: Instructions + Tools + Response Format
2. **Tools enable capabilities**: Web search, code execution, MCP servers
3. **Workflows orchestrate**: Edges define communication paths
4. **Structured outputs**: Enable reliable routing and parsing
5. **Azure integration**: Enterprise-scale deployment with monitoring
6. **Spec-kit accelerates**: AI implements from specifications

### **🚀 Next Steps**

Continue your agent-framework journey:

- **Add more agents**: Create a Marketing Specialist or Risk Management Agent using spec-kit
- **Custom MCP tools**: Build domain-specific tool servers
- **Advanced workflows**: Implement conditional branching and loops
- **Production features**: Add authentication, rate limiting, cost tracking

### **📚 Resources**

- **Agent Framework**: [github.com/microsoft/agent-framework](https://github.com/microsoft/agent-framework)
- **Microsoft Foundry**: [ai.azure.com](https://ai.azure.com)
- **Spec-Kit**: [github.com/spec-kit](https://github.com/spec-kit)

**Happy coding at Microsoft Ignite 2025!**