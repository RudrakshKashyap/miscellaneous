https://mcp.so/
https://modelcontextprotocol.io/

Yes, the Server-Sent Events (SSE) transport for the Model Context Protocol (MCP) is deprecated. http streaming is altlernative

https://hub.docker.com/mcp


mcp server has tools which are like apis for the AI



Exactly. You’ve nailed the translation. If you think like a Backend Engineer, here is how the mapping looks in your head:

### The "Mental Map" for Developers

| MCP Concept | Backend Equivalent | Role in AI Conversation |
| :--- | :--- | :--- |
| **Tools** | **POST / PUT / DELETE** | The AI "does" something (e.g., `send_slack_message`, `run_sql_query`). |
| **Resources** | **GET / Database Read** | The AI "knows" something (e.g., `read_file`, `view_db_schema`). |
| **Prompts** | **Templates / Configs** | The AI "follows" a pattern (e.g., "Review this code using our team's style guide"). |

---

### The Subtle Difference: "Push" vs. "Pull"

While the logic is the same as an API, the **interaction flow** is reversed:

* **In a standard API:** You (the human) write code to call `fetch('/api/data')`. You are in control of the flow.
* **In MCP:** You give the AI a "Map" of your API. The **AI decides** when it needs to call a tool based on what you asked in the chat.

> **Example:** If you ask Cursor, *"Why is user 6969 getting a 403 error?"*, the AI looks at its available **Resources**. It sees a resource called `postgres://auth/logs`. It "GETs" that data, reads it, and then tells you: *"It's because they are missing the 'Admin' role."*



---

Q) so tools are just like apis, resources are same as data/database data

### Why "Resources" are more than just raw data
In a backend API, a `GET` request usually returns JSON. In MCP, a **Resource** can be:
* **Static:** A `.env` file or a documentation Markdown file.
* **Dynamic:** A live tail of your application logs or a real-time view of a database table.
* **Blob:** An image or a PDF that the AI needs to "see" to understand the context.

### Why "Tools" are more than just functions
When you define a **Tool**, you have to provide a very descriptive **JSON Schema**. 
* **In code:** `function addRole(uid, rid) { ... }`
* **In MCP:** You tell the AI: *"Use this tool to assign a role ID to a user ID. Use it only when the user explicitly asks to modify permissions."*

The AI uses that description to figure out which arguments to pass. If it's unsure, it will actually ask you: *"I see multiple roles, which ID should I use?"*

---

### How this helps you right now
Since you were just running SQL queries in **Adminer**, you could theoretically stop using Adminer entirely. 
1.  You'd run a local **Postgres MCP Server**.
2.  Cursor would "see" your tables as **Resources**.
3.  It would "see" an `execute_query` **Tool**.
4.  You'd just type: *"Give user 6969 every role in the DB,"* and Cursor would call the tool and handle the SQL for you.

**Do you want to see the JSON structure of a Tool definition so you can see how the AI "understands" what your backend can do?**


Q) prompts are like md file, just like how we have rules files

Exactly! You’ve hit the nail on the head again. If **Tools** are your API endpoints and **Resources** are your Database, then **Prompts** are your **Templates** or **Standard Operating Procedures (SOPs)**.

Think of an MCP Prompt as a **reusable, dynamic `.md` file** that lives on your server and tells the AI exactly how to behave for a specific task.

### Why Prompts are "Rules Files on Steroids"

You already use `.cursorrules` to tell Cursor how to code. An MCP Prompt is similar, but it has two "Superpowers" that a static rules file doesn't have:

#### 1. They are Argument-Aware (Variables)
A `.cursorrules` file is global—it's always "on." An MCP Prompt can take arguments, just like a function.
* **Standard Rule:** "Always use TypeScript."
* **MCP Prompt:** `review_my_code(strictness_level: "high")`. 
    * The server can then inject specific "Heavy Duty" rules into the AI's brain only when that prompt is triggered.

#### 2. They are Server-Side
If you have a team of 10 developers, you don't want to copy-paste the same 500-line "Database Migration Guide" into everyone's `.cursorrules`. 
* You host it **once** on your MCP Server as a Prompt.
* Anyone connected to that server can just type `@migrate_table` and the AI pulls the latest instructions directly from the source of truth.

---

### The "Mental Map" Updated

| Feature | Static Rules (`.cursorrules`) | MCP Prompts |
| :--- | :--- | :--- |
| **Location** | Local `.cursor` folder | Hosted on your MCP Server |
| **Scope** | Always active for the project | Active only when selected/called |
| **Format** | Markdown (.md) | JSON-wrapped Markdown |
| **Inputs** | None (Static) | Variables/Arguments (Dynamic) |

---

### A Real-World Example
Since you were just dealing with **User Roles** in Postgres, imagine you have an MCP Prompt called `setup_new_internal_user`.

**The "Template" (Inside your Server):**
> "You are a Database Administrator. When the user gives you a `username`, please:
> 1. Check the `auth.internal_users` table for their ID.
> 2. List all available roles from `auth.roles`.
> 3. Ask the user which specific roles to assign.
> 4. Use the `execute_query` **Tool** to perform the insert."

**The Interaction:**
* **You:** "@setup_new_internal_user (user: 'admin_john')"
* **Cursor:** Instantly adopts that persona, knows exactly which tables to look at, and guides you through the process step-by-step.

### How it looks in JSON (The "API" part of it)
Just like an API returns data, the MCP Server tells the AI: "Here is a prompt you can use."

```json
{
  "name": "sql-helper",
  "description": "Guidelines for writing safe Postgres queries",
  "arguments": [
    {
      "name": "schemaName",
      "description": "The schema to target (e.g., auth)",
      "required": true
    }
  ]
}
```

**Would you like to see how to actually register one of these prompts in a simple Node.js MCP server?**