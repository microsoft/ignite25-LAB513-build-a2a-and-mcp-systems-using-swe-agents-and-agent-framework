# Build A2A and MCP Systems using SWE Agents and agent-framework

## Lab Scenario

In this lab, you'll build a multi-agent event planning system using **[Microsoft Agent Framework](https://github.com/microsoft/agent-framework)**, an enterprise-ready framework that combines the best of Semantic Kernel and AutoGen. The framework's key differentiator is its **workflow orchestration capabilities**, which enable you to define complex multi-agent interactions as declarative workflows with clear execution paths, state management, and observability.

You'll create specialized AI agents that collaborate to plan comprehensive events, learning how to:

- **Build Multi-Agent Workflows**: Orchestrate multiple specialized agents (Event Coordinator, Venue Specialist, Budget Analyst, Catering Coordinator, and Logistics Manager) that work together through workflow edges and message passing. The framework supports **Agent-to-Agent (A2A) protocol** for direct inter-agent communication and **Model Context Protocol (MCP)** for integrating external tools like weather forecasting and calendar management.

- **Deploy to Azure AI Foundry**: Run your agent-framework workflows in Azure AI Foundry, where agents are automatically registered and managed with full enterprise-grade capabilities, security, and observability.

- **Visualize with DevUI and Azure AI Foundry**: Monitor real-time agent interactions, workflow execution graphs, and message flows through interactive interfaces both locally (DevUI) and in the cloud (Azure AI Foundry).

The system demonstrates concurrent workflow execution patterns where agents work in sequence and in parallel, exchange information through the workflow, invoke MCP tools for specialized capabilities, and synthesize comprehensive event plans. You'll also implement **human-in-the-loop** capabilities, allowing user input and approval at critical decision points during agent execution.

!IMAGE[Event Planning Agent Design Final 1.png](instructions310255/Event Planning Agent Design Final 1.png)

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

### **Agent-to-Agent (A2A) Protocol**

A2A is an emerging standard for enabling direct communication between AI agents. With A2A:

- **Direct Communication**: Agents can message each other directly, reducing latency and enabling more natural collaboration
- **Standardized Messaging**: A common protocol ensures agents built with different frameworks can communicate
- **Framework Compatibility**: Microsoft Agent Framework supports A2A for inter-agent communication patterns

Your workflow in this lab uses message passing between agents through workflow edges. Agent Framework also supports A2A protocol for scenarios requiring direct agent-to-agent communication beyond workflow orchestration.

---


===


## 1.  Lab Set-Up & Sign-In to Skillable Events GitHub

Sign in to your lab environment using:
- Username: +++@lab.VirtualMachine(Win11-Pro-Base).Username+++
- Password: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++

Your lab environment comes pre-configured with:

- **Visual Studio Code** - Primary development environment
- **Python 3.13** - Latest stable version
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

**Open *src/spec-to-agents* folder from root in VS Code Explorer** and examine this structure:

```
spec-to-agents/
├── agents/                    # Agent definitions (one per specialist)
│   ├── venue_specialist.py
│   ├── budget_analyst.py
│   ├── catering_coordinator.py
│   ├── logistics_manager.py
│   └── event_coordinator.py
├── prompts/                   # System prompts (agent instructions)
│   └── [matching agent files]
├── tools/                     # MCP tools and @ai_functions
│   ├── bing_search.py
│   ├── weather.py
│   └── calendar.py
├── models/                    # Data models for structured outputs
│   └── messages.py
├── workflow/                  # Workflow orchestration logic
│   ├── core.py               # 🎯 Main workflow builder
│   └── executors.py          # Custom executor logic
└── console.py                # CLI entry point for testing
```

> [!knowledge] **Agent Framework Code Organization**
> 
> Agent Framework follows a clear separation of concerns:
> - **`/agents/`**: Agent creation functions that combine prompts + tools
> - **`/prompts/`**: System instructions defining agent behavior and expertise
> - **`/tools/`**: Reusable capabilities (web search, weather, calendars)
> - **`/workflow/`**: Orchestration logic connecting agents with edges
> - **`/models/`**: Type-safe data structures for agent communication

---
===

## 6. Exercise 1: Create Your First Agent

**Concept**: In agent-framework, an **agent** is created by combining three elements:
1. **System instructions** (the "personality" and expertise)
2. **Tools** (capabilities like web search or code execution)
3. **Response format** (structured output for predictable responses)

**Locate** the file **`agents/venue_specialist.py`** and find the `# TODO: Exercise 1` comment (around line 10).

**Replace the TODO section with**:

```python
# TODO: Exercise 1 - Create a Venue Specialist agent with web search capability

def create_agent(client: AzureAIAgentClient, mcp_tool: MCPStdioTool | None) -> ChatAgent:
    """
    Create Venue Specialist agent for event planning workflow.
    
    This agent uses web search to find and evaluate venue options based on
    event requirements like capacity, location, and budget.
    """
    tools = [web_search]  # Add web search capability
    
    if mcp_tool is not None:
        tools.append(mcp_tool)  # Add sequential thinking for complex reasoning
    
    return client.create_agent(
        name="VenueSpecialist",
        instructions=venue_specialist.SYSTEM_PROMPT,  # From prompts/venue_specialist.py
        tools=tools,
        response_format=SpecialistOutput,  # Ensures structured JSON responses
        store=True,  # Enable conversation history storage
    )
```

> [!knowledge] **What You Just Learned**
> 
> **Agent Creation Pattern**:
> - `client.create_agent()` is the factory method for all agents
> - `name`: Identifies the agent in workflow traces
> - `instructions`: System prompt defining agent expertise (from `/prompts/`)
> - `tools`: List of capabilities the agent can invoke
> - `response_format`: Pydantic model enforcing structured output
> - `store=True`: Enables **service-managed threads** - Azure automatically maintains conversation history so you don't have to manually track messages
> 
> **Key Insight**: The agent doesn't "know" about venues - it gains that knowledge by using the `web_search` tool to research options in real-time.

---

===

## 7. Exercise 2: Implement a Web Search Tool

**Concept**: Tools in agent-framework are Python functions decorated with `@ai_function`. The LLM can discover and invoke these tools automatically when needed.

!IMAGE[Agent Tools Final.png](instructions310255/Agent Tools Final.png)

**Open** **`tools/bing_search.py`** and find the `# TODO: Exercise 2` comment (around line 15).

**Replace the TODO section with**:

```python
# TODO: Exercise 2 - Implement web search as an @ai_function tool

@ai_function  # This decorator makes the function discoverable by agents
async def web_search(
    query: Annotated[str, Field(description="Search query to find information on the web")],
) -> str:
    """
    Search the web using Bing Search API and return formatted results.
    
    The LLM receives:
    - Number of results found
    - Title, snippet, and URL for each result
    - Formatted for easy parsing and citation
    """
    try:
        # Create temporary agent with Bing tool for the search
        web_search_tool = HostedWebSearchTool(
            description="Search the web for current information using Bing"
        )
        
        async with create_agent_client() as client:
            agent = client.create_agent(
                name="BingWebSearchAgent",
                tools=[web_search_tool],
                system_message="You are a web search agent using Bing.",
                tool_choice=ToolMode.REQUIRED(function_name="web_search"),
                store=True,
                model_id=os.getenv("WEB_SEARCH_MODEL", "gpt-4o-mini"),
            )
            response = await agent.run(f"Perform a web search for: {query}")
            return response.text
            
    except Exception as e:
        return f"Error performing web search: {type(e).__name__} - {str(e)}"
```

> [!knowledge] **What You Just Learned**
> 
> **@ai_function Pattern**:
> - `@ai_function`: Decorator that generates OpenAI function schemas automatically
> - `Annotated[type, Field(...)]`: Provides descriptions for the LLM to understand parameters
> - **Async functions**: All tools should be async for non-blocking I/O operations
> - **Error handling**: Always return string results, even for errors (LLMs can interpret error messages)
> 
> **HostedWebSearchTool**:
> - Built-in Azure AI tool - Grounding with Bing Search
> - Automatically handles API authentication via Azure credentials
> - Returns structured results with source citations
> 
> **When is this called?**: When the Venue Specialist (or any agent with this tool) needs to find venues online, the LLM will automatically invoke `web_search("event venues in Seattle for 50 people")`.

---

===

## 8. Exercise 3: Add MCP Sequential Thinking Tool

**Concept**: **MCP (Model Context Protocol)** is an open standard for connecting AI models to external tools. The `sequential-thinking-tools` MCP server provides advanced reasoning capabilities.

**Open** **`tools/mcp_tools.py`** and find the `# TODO: Exercise 3` comment (around line 8).

**Replace the TODO section with**:

```python
# TODO: Exercise 3 - Create MCP sequential thinking tool for complex reasoning

def create_sequential_thinking_tool() -> MCPStdioTool:
    """
    Create sequential-thinking-tools MCP server for breaking down complex tasks.
    
    This tool helps agents:
    - Decompose complex planning into steps
    - Track reasoning through multi-step problems
    - Maintain context across reasoning chains
    
    Returns unconnected MCPStdioTool - must be used with async context manager.
    """
    return MCPStdioTool(
        name="sequential-thinking-tools",
        command="npx",  # Use Node.js package runner
        args=["-y", "mcp-sequentialthinking-tools"],  # Install and run MCP server
        env={
            "MAX_HISTORY_SIZE": os.getenv("MAX_HISTORY_SIZE", "1000"),
        },
    )
```

> [!knowledge] **What You Just Learned**
> 
> **MCP (Model Context Protocol)**:
> - Open standard for AI model ↔ tool communication
> - Similar to how USB is a standard for device connections
> - Enables language models to discover and use external tools
> 
> **MCPStdioTool**:
> - Connects to MCP servers via stdio (standard input/output)
> - `command` and `args`: Launches the MCP server process
> - `npx -y mcp-sequentialthinking-tools`: Downloads and runs the MCP server on-demand
> 
> **Sequential Thinking**:
> - Helps agents break down complex decisions: "Should I pick Venue A or B?"
> - Creates step-by-step reasoning chains
> - Example: Budget Analyst uses this to compare allocation strategies
> 
> **Async Context Manager Pattern**:
> ```python
> async with create_sequential_thinking_tool() as mcp_tool:
>     # MCP server starts automatically
>     # Tool is now connected and ready
>     pass
>     # Server stops automatically when exiting context
> ```

---

===

## 9. Exercise 4: Define Structured Output Format

**Concept**: **Structured outputs** ensure agents return predictable, parseable responses instead of free-form text. This enables reliable workflow routing.

**Open** **`models/messages.py`** and find the `# TODO: Exercise 4` comment (around line 25).

**Replace the TODO section with**:

```python
# TODO: Exercise 4 - Define SpecialistOutput for structured agent responses

class SpecialistOutput(BaseModel):
    """
    Structured output from each specialist agent.
    
    This model enforces that specialists provide:
    1. A concise summary of their work
    2. Routing decision (next_agent or user_input_needed)
    3. Optional user prompt if input is needed
    
    Example responses:
    
    Route to next agent:
    {
        "summary": "Researched 3 venues: The Foundry ($2k, 60 capacity), 
                    Pioneer Square Hall ($1.5k, 50 capacity), 
                    Fremont Studios ($3k, 80 capacity). 
                    Recommended The Foundry for best value.",
        "next_agent": "budget",
        "user_input_needed": false,
        "user_prompt": null
    }
    
    Request user input:
    {
        "summary": "Found 3 excellent venue options with different tradeoffs.",
        "next_agent": null,
        "user_input_needed": true,
        "user_prompt": "Which venue do you prefer: The Foundry (modern, $2k), 
                        Pioneer Square Hall (historic, $1.5k), or 
                        Fremont Studios (industrial, $3k)?"
    }
    """
    
    summary: str = Field(
        description="Concise summary of this specialist's recommendations (max 200 words)"
    )
    
    next_agent: str | None = Field(
        description=(
            "ID of next agent to route to ('venue', 'budget', 'catering', 'logistics'), "
            "or null if done/need user input"
        )
    )
    
    user_input_needed: bool = Field(
        default=False,
        description="Whether user input is required before proceeding"
    )
    
    user_prompt: str | None = Field(
        default=None,
        description="Question to ask user if user_input_needed=True"
    )
```

> [!knowledge] **What You Just Learned**
> 
> **Structured Outputs with Pydantic**:
> - `BaseModel`: Pydantic base class for type-safe data models
> - `Field(description=...)`: Provides hints to the LLM about each field's purpose
> - **Type hints**: `str | None` means "string or null" - LLM understands optional fields
> - **Default values**: `default=False` provides fallback if field is omitted
> 
> **Why This Matters**:
> - **Predictable parsing**: You can reliably extract `next_agent` for routing
> - **No regex parsing**: No need to parse free-form text like "I think we should move to the budget analyst next..."
> - **LLM compliance**: When you set `response_format=SpecialistOutput`, the LLM is forced to return valid JSON matching this schema
> 
> **Routing Logic**:
> ```python
> if output.user_input_needed:
>     # Pause workflow for human input
> elif output.next_agent:
>     # Route to specified agent
> else:
>     # Workflow complete - synthesize final plan
> ```

---

===

## 10. Exercise 5: Build the Workflow with Edges

**Concept**: The **WorkflowBuilder** defines how agents communicate through **edges**. Each edge represents a potential message-passing path between executors.

**Open** **`workflow/core.py`** and find the `# TODO: Exercise 5` comment (around line 85).

**Replace the TODO section with**:

```python
# TODO: Exercise 5 - Build the event planning workflow with agent executors

def build_event_planning_workflow(
    client: AzureAIAgentClient,
    mcp_tool: MCPStdioTool | None = None,
) -> Workflow:
    """
    Build the multi-agent event planning workflow.
    
    Architecture: Star topology with Event Coordinator at the center
    
    Flow:
    1. User → Coordinator (analyzes request)
    2. Coordinator → Venue Specialist (finds venues)
    3. Venue → Coordinator (reports findings)
    4. Coordinator → Budget Analyst (allocates budget)
    5. Budget → Coordinator (reports allocation)
    6. Coordinator → Catering (plans menu)
    7. Catering → Coordinator (reports menu)
    8. Coordinator → Logistics (creates timeline)
    9. Logistics → Coordinator (reports timeline)
    10. Coordinator → User (synthesizes final plan)
    """
    
    # Create all agents
    coordinator_agent = event_coordinator.create_agent(client)
    venue_agent = venue_specialist.create_agent(client, mcp_tool)
    budget_agent = budget_analyst.create_agent(client, mcp_tool)
    catering_agent = catering_coordinator.create_agent(client, mcp_tool)
    logistics_agent = logistics_manager.create_agent(client, mcp_tool)
    
    # Create coordinator executor with custom routing logic
    coordinator = EventPlanningCoordinator(coordinator_agent)
    
    # Wrap specialist agents as AgentExecutors
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
                "catering, and logistics coordination. Supports human-in-the-loop."
            ),
            max_iterations=30,  # Safety limit to prevent infinite loops
        )
        # Set coordinator as starting point
        .set_start_executor(coordinator)
        
        # Bidirectional edges: Coordinator ↔ Each Specialist
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
    
    # Set stable ID for DevUI
    workflow.id = "event-planning-workflow"
    
    return workflow
```

> [!knowledge] **What You Just Learned**
> 
> **Workflow Builder Pattern**:
> - **`WorkflowBuilder`**: Abstraction for constructing workflows
> - **`.set_start_executor()`**: Defines workflow entry point
> - **`.add_edge(from, to)`**: Adds directed communication path
> - **`.build()`**: Validates and creates the workflow instance
> 
> **Executor Types**:
> - **`AgentExecutor`**: Standard wrapper for ChatAgent instances
> - **`EventPlanningCoordinator`**: Custom executor with routing logic
> - **`id` parameter**: Used for routing (`next_agent="budget"` routes to executor with `id="budget"`)
> 
> **Star Topology** (vs. Linear):
> ```
> Linear:  A → B → C → D (rigid, single path)
> 
> Star:      B
>           ↗ ↘
>     A ← → Hub ← → C    (flexible, coordinator decides routing)
>           ↖ ↙
>            D
> ```
> 
> **Bidirectional Edges**:
> - `.add_edge(coordinator, venue)`: Coordinator can send to venue
> - `.add_edge(venue, coordinator)`: Venue can send back to coordinator
> - **Why?** Coordinator reads `SpecialistOutput.next_agent` and routes dynamically
> 
> **Workflow Execution**:
> ```python
> result = await workflow.run("Plan a party for 50 people")
> # 1. Starts at coordinator
> # 2. Coordinator routes to venue
> # 3. Venue returns SpecialistOutput(next_agent="budget")
> # 4. Coordinator routes to budget
> # 5. Budget returns SpecialistOutput(next_agent="catering")
> # ... and so on
> ```

---
===

## 11. Install Dependencies and Environment Setup

Before running the workflow, let's install the project dependencies.

1. **Ensure you are in the repository root directory**:
   ```powershell
   cd $HOME\spec-to-agents  # or wherever you cloned the repo
   ```

2. **Install dependencies** (includes automatic virtual environment creation):
   ```powershell
   $env:GIT_LFS_SKIP_SMUDGE = "1"; uv sync --extra dev
   ```

   > [!knowledge] **What does this do?**
   > 
   > - `GIT_LFS_SKIP_SMUDGE = "1"`: Skips downloading large Git LFS files (not needed for this lab)
   > - `uv sync --extra dev`: Creates a `.venv` automatically and installs all dependencies
   > - `--extra dev`: Includes development tools like linters and formatters
   > 
   > **Note**: You don't need to manually activate the virtual environment - `uv` handles this automatically when you run `uv run` commands!

3. **Verify `.env` file was created** (from earlier `azd provision`):
   
   Check that **`.env`** exists in the root directory with values like:
   ```
   AZURE_OPENAI_ENDPOINT=https://...
   PROJECT_CONNECTION_STRING=...
   AGENTS_MODEL_DEPLOYMENT_NAME=gpt-4o
   ```
   
   If missing, manually run from the repository root:
   ```powershell
   .\scripts\generate-settings.ps1
   ```

---
===

## 12. Run Your First Multi-Agent Workflow

Now let's see your agents in action!

1. **Start the console application** (from the repository root):
   ```powershell
   uv run console
   ```

2. **You'll see a welcome screen** with agent descriptions and MCP tool initialization:
   
   ```
   ╭─────────────────────────────────────────────────────────────────────╮
   │                                                                     │
   │                      Event Planning Workflow                        │
   │              Interactive CLI with AI-Powered Agents                 │
   │                                                                     │
   ╰─────────────────────────────────────────────────────────────────────╯

   This workflow uses specialized AI agents to help you plan events:
     • Venue Specialist - Researches and recommends venues
     • Budget Analyst - Manages costs and financial planning
     • Catering Coordinator - Handles food and beverage
     • Logistics Manager - Coordinates schedules and resources

   You may be asked for clarification or approval at various steps.

   Available tools: [ 'sequentialthinking_tools' ]
   Sequential Thinking MCP Server running on stdio
   ✓ Workflow loaded successfully
   ```

3. **Enter a test prompt** when asked. Try one of these examples:
   
   ```
   ─────────────────────────── Event Planning Request ───────────────────────────

   Enter your event planning request
   Or select from these examples:
     1. Plan a corporate holiday party for 50 people, budget $5000
     2. Organize a wedding reception for 150 guests in Seattle
     3. Host a tech conference with 200 attendees, need catering and AV

   Your request (or 1-3 for examples): _
   ```
   
   **Option 1** (Simple with clear constraints):
   ```
   Plan a corporate holiday party for 50 people, budget $5000
   ```
   
   **Option 2** (Wedding - larger scale):
   ```
   Organize a wedding reception for 150 guests in Seattle
   ```
   
   **Option 3** (Complex multi-day event):
   ```
   Host a tech conference with 200 attendees, need catering and AV
   ```
   
   Or type your own custom event planning request!

4. **Watch the workflow execute**. You'll see structured output as agents work sequentially:

   ```
   ─────────────────────────────── Workflow Execution ───────────────────────────


   ──────────────────────────────────── venue ────────────────────────────────────
   {"summary":"I recommend The Foundry in Seattle for your wedding reception with 
   150 guests. It offers a spacious capacity suitable for this size, located 
   centrally with excellent accessibility including parking and public transit 
   options. The venue provides elegant ambiance appropriate for weddings and is 
   equipped with essential amenities such as AV equipment, catering facilities, 
   and accessibility features.","next_agent":"budget","user_input_needed":false,
   "user_prompt":null}

   ──────────────────────────────────── budget ───────────────────────────────────
   {"summary":"For your wedding reception of 150 guests at The Foundry in Seattle, 
   I recommend a total budget around $22,500 based on industry standards for formal 
   events at $150 per person. The budget allocation would be: Venue rental 
   approximately $11,000 (about 49%), Catering $6,750 (30%), Equipment/AV $1,350 
   (6%), Decorations $900 (4%), Staff/Services $675 (3%), and a contingency fund 
   of $1,800 (8%).","next_agent":"catering","user_input_needed":false,
   "user_prompt":null}

   ─────────────────────────────────── catering ──────────────────────────────────
   {"summary":"For the wedding reception with 150 guests in Seattle, I recommend 
   a buffet-style catering menu to balance formality and flexibility. The menu 
   includes three entrée options (e.g., chicken, beef, vegetarian), two sides, 
   a fresh salad, assorted desserts, and beverage options including wine, beer, 
   and non-alcoholic drinks. This approach accommodates vegetarian and gluten-free 
   guests by default. Estimated cost is around $45-$50 per person.","next_agent":
   "logistics","user_input_needed":false,"user_prompt":null}

   ─────────────────────────────────── logistics ─────────────────────────────────
   ╭──────────────────────────────── Function Call ─────────────────────────────╮
   │ 🔧 Tool Call: create_calendar_event                                        │
   │ Call ID: ["run_zWZDBOHAgp7PlIoJDi1Xc1tA", "call_JbFWLm2ebkzstYsugJdVUIsH"]│
   │                                                                            │
   │ {"event_title": "Wedding Reception - The Foundry, Seattle",               │
   │  "start_date": "2024-09-21", "start_time": "17:00", "duration_hours": 5,  │
   │  "location": "The Foundry, Seattle", "description": "Wedding reception    │
   │  for 150 guests. Setup starts at 2:00 PM, event from 5:00 PM to 10:00 PM"}│
   ╰────────────────────────────────────────────────────────────────────────────╯

   ╭──────────────────────────────── Function Call ─────────────────────────────╮
   │ 🔧 Tool Call: get_weather_forecast                                         │
   │ Call ID: ["run_zWZDBOHAgp7PlIoJDi1Xc1tA", "call_ThfrWG6ElVqYZCqFpxWTm8Aq"]│
   │                                                                            │
   │ {"location": "Seattle", "days": 1}                                         │
   ╰────────────────────────────────────────────────────────────────────────────╯

   ╭───────────────────────────────── Tool Result ──────────────────────────────╮
   │ Call ID: ["run_zWZDBOHAgp7PlIoJDi1Xc1tA", "call_JbFWLm2ebkzstYsugJdVUIsH"]│
   │                                                                            │
   │ Successfully created event 'Wedding Reception - The Foundry, Seattle' on  │
   │ 2024-09-21 at 17:00 in calendar 'event_planning'                          │
   ╰────────────────────────────────────────────────────────────────────────────╯

   ╭───────────────────────────────── Tool Result ──────────────────────────────╮
   │ Call ID: ["run_zWZDBOHAgp7PlIoJDi1Xc1tA", "call_ThfrWG6ElVqYZCqFpxWTm8Aq"]│
   │                                                                            │
   │ Weather forecast for Seattle, United States:                              │
   │ 2025-11-05: moderate rain, 10.2°C to 14.0°C, 88% chance of precipitation  │
   ╰────────────────────────────────────────────────────────────────────────────╯

   {"summary":"Event timeline for the wedding reception at The Foundry in 
   Seattle on September 21, 2024: Setup begins at 2:00 PM to allow for catering, 
   decoration, and AV preparation. Guest arrival and doors open at 5:00 PM, with 
   buffet service starting around 5:30 PM lasting approximately two hours, 
   followed by speeches and dancing. Cleanup concludes by 10:00 PM. Weather 
   forecast indicates moderate rain with temperatures between 10.2°C and 14.0°C, 
   so an indoor venue plan is appropriate.","next_agent":null,
   "user_input_needed":false,"user_prompt":null}
   ```

   > [!knowledge] **Understanding the Output**
   > 
   > **Agent Headers**: Each `────── agent_name ──────` shows which specialist is working
   > 
   > **Structured Output**: The JSON you see is the `SpecialistOutput` model:
   > - `"summary"`: What the agent accomplished
   > - `"next_agent"`: Where to route next ("budget", "catering", "logistics", or null)
   > - `"user_input_needed"`: Whether to pause for human input
   > 
   > **Tool Calls**: When agents invoke tools, you see:
   > - Function name and parameters (what the agent is requesting)
   > - Tool results (what the tool returned)
   > 
   > **Example**: The Logistics Manager calls `create_calendar_event` and `get_weather_forecast` in parallel, then incorporates the results into its plan.

5. **Human-in-the-Loop Interaction** (if needed):
   
   Some agents may pause and request your input. You'll see a formatted prompt:
   
   ```
   ╭────────────────────────────────────────────────────────────────────╮
   │                    🤔 VENUE Agent Request                          │
   │                                                                    │
   │ Type: selection                                                    │
   │                                                                    │
   │ Question:                                                          │
   │ I found 3 excellent venue options. Which do you prefer?           │
   │                                                                    │
   │ Additional Context:                                                │
   │ venues:                                                            │
   │   • The Foundry: $2k, modern, downtown                            │
   │   • Pioneer Square: $1.8k, historic, transit access               │
   │   • Fremont Studios: $2.5k, industrial, rooftop                   │
   ╰────────────────────────────────────────────────────────────────────╯
   
   Your response: _
   ```
   
   Type your choice (e.g., "The Foundry" or "1") and press Enter to continue.

6. **Final Output**:
   
   After all agents complete, you'll see the comprehensive event plan:
   
   ```
   ──────────────────────────────── Final Event Plan ─────────────────────────────

   ╭──────────────────────────────── Event Plan Summary ───────────────────────╮
   │                                                                            │
   │                             Executive Summary                              │
   │                                                                            │
   │  This comprehensive plan outlines the wedding reception for 150 guests in │
   │  Seattle, designed to ensure a seamless and memorable event.              │
   │                                                                            │
   │                                  Venue                                     │
   │                                                                            │
   │  The Foundry in Seattle is chosen for its spacious capacity, elegant      │
   │  ambiance, central location, and excellent amenities, including AV,       │
   │  catering kitchen, and accessibility. It comfortably accommodates 150     │
   │  guests with versatile event space.                                       │
   │                                                                            │
   │                                 Budget                                     │
   │                                                                            │
   │  The budget estimate is approximately $22,500, allocated roughly as 49%   │
   │  for venue rental ($11,000), 30% for catering ($6,750), 6% for AV         │
   │  equipment ($1,350), 4% for decorations ($900), 3% for staff/services     │
   │  ($675), and 8% contingency ($1,800). This balanced distribution ensures  │
   │  quality and flexibility.                                                 │
   │                                                                            │
   │                                Catering                                    │
   │                                                                            │
   │  A buffet-style menu is planned with three entrée options (chicken, beef, │
   │  vegetarian), two sides, salad, assorted desserts, and beverage selections│
   │  including wine, beer, and non-alcoholic drinks. This setup accommodates  │
   │  dietary preferences including vegetarian and gluten-free guests and      │
   │  supports a social, inviting atmosphere.                                  │
   │                                                                            │
   │                               Logistics                                    │
   │                                                                            │
   │  Event timeline includes setup starting at 2 PM with coordination of      │
   │  decorators, AV, and caterers. Guests arrive by 5 PM with buffet service  │
   │  commencing shortly after. Activities like speeches and dancing will      │
   │  follow, concluding at 10 PM. Weather forecast calls for moderate rain,   │
   │  so the indoor venue is ideal. The event has been calendared for smooth   │
   │  coordination.                                                             │
   │                                                                            │
   │                              Next Steps                                    │
   │                                                                            │
   │  Confirm final guest list and menu choices, finalize vendor contracts for │
   │  decorations and AV, and communicate timeline clearly to all service      │
   │  providers and wedding party.                                             │
   │                                                                            │
   │  This integrated plan provides a strong foundation for a successful       │
   │  wedding reception aligned with client expectations and practical         │
   │  considerations.                                                           │
   │                                                                            │
   ╰────────────────────────────────────────────────────────────────────────────╯

   ✓ Event planning complete!
   ```

> [!knowledge] **What Just Happened?**
> 
> **Workflow Execution Flow**:
> 1. **Venue Specialist**: Researched venues in Seattle for 150 guests, selected The Foundry, returned `next_agent="budget"`
> 2. **Event Coordinator**: Read the routing decision, forwarded venue details to Budget Analyst
> 3. **Budget Analyst**: Calculated $22,500 budget based on $150/person industry standard, allocated across categories, returned `next_agent="catering"`
> 4. **Event Coordinator**: Routed to Catering with budget constraints ($6,750 for food)
> 5. **Catering Coordinator**: Designed buffet menu at $45-50/person, included dietary accommodations, returned `next_agent="logistics"`
> 6. **Event Coordinator**: Routed to Logistics with full context
> 7. **Logistics Manager**: 
>    - Invoked `create_calendar_event` tool → Created calendar entry for Sept 21
>    - Invoked `get_weather_forecast` tool → Retrieved Seattle weather (moderate rain)
>    - Created detailed timeline based on venue, catering, and weather
>    - Returned `next_agent=null` (workflow complete)
> 8. **Event Coordinator**: Detected completion, synthesized comprehensive final plan from all agent outputs
> 
> **Key Observations**:
> - **Service-managed threads**: Each agent automatically remembered all prior context
> - **Structured routing**: The `next_agent` field drove the workflow path
> - **Autonomous tool use**: Agents decided when to invoke tools (you didn't specify)
> - **Parallel tool calls**: Logistics Manager called calendar and weather tools simultaneously
> - **No human input needed**: Workflow ran autonomously because all constraints were provided


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

   !IMAGE[devui-run-workflow.png](instructions310255/devui-run-workflow.png)

3. **Watch Execution**:
   - **Graph**: Nodes light up green as agents execute
   - **Events Tab**: Real-time agent outputs and structured JSON responses
   - **Traces Tab**: Execution duration and routing decisions
   - **Tools Tab**: Tool calls (like `web_search`) with arguments and results

   !IMAGE[devui-execution.png](instructions310255/devui-execution.png)

> [!knowledge] **DevUI vs Console**
> 
> - **Visual workflow graph**: See agent communication patterns
> - **Detailed traces**: Debug with execution timings and message routing
> - **Tool inspection**: View all tool calls and results in one place


---
===
## 14. BONUS: Explore Supervisor Pattern (Alternative Branch)

The main branch uses a **star topology** where a coordinator routes between specialists. There's an alternative **supervisor pattern** available on a different branch.

!IMAGE[Event Planning Agent Design Bonus.png](instructions310255/Event Planning Agent Design Bonus.png)

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