# IFS MCP Skills

A community-maintained collection of skills for the [IFS MCP Server](https://github.com/knakit/ifs-mcp), enabling Claude to interact with IFS Cloud using plain business language.

---

## What Are Skills?

Skills are markdown files that Claude loads dynamically to improve performance on specialized tasks. Each skill in this repo teaches Claude how to use a specific IFS Cloud API projection — what endpoints exist, which fields matter, how to filter results, and what the business terminology means.

This means you can ask Claude things like:

> *"Show me all open customer orders for CUST-001"*
> *"Create a new customer in IFS with this information..."*
> *"Run the aged receivables quick report for last month"*

...without needing to know the underlying OData syntax or field names.

---

## Available Skills

| Skill | Module | Description |
|-------|--------|-------------|
| [ifs-sales-customer.md](skills/ifs-sales-customer.md) | Sales | Create, search, update customers, addresses, contacts, credit info |
| [ifs-reporting-quick-reports.md](skills/ifs-reporting-quick-reports.md) | Reporting | Search, parameterise, and execute IFS Quick Reports |

---

## Using Skills

### Claude Desktop / Claude.ai

Add skill files to your Claude project instructions, or paste the contents directly into a conversation alongside your IFS MCP Server connection.

### Manual

Copy the contents of any skill file and include it in your system prompt or project context when using the IFS MCP Server.

---

## Skill Structure

Each skill file is plain markdown — no special syntax required:

```markdown
# IFS [Area] API Guide

Use the `call_protected_api` tool with the endpoints below.

## 1. Operation Name

Plain language description of what this does.

**Endpoint:**
```
/main/ifsapplications/projection/v1/SomeHandling.svc/SomeSet
```

**Example:**
```
/main/ifsapplications/projection/v1/SomeHandling.svc/SomeSet?$filter=...&$top=10
```
```

Skills are intentionally written in **business language** — they map plain terms to API fields so you don't have to.

---

## Module Taxonomy

Skills follow the naming convention `ifs-[module]-[area].md`:

| Module | Covers |
|--------|--------|
| `common` | Shared references — OData syntax, field types, enumerations |
| `reporting` | IFS report related skills, quick reports, order reports, report archive etc. |
| `sales` | Customer orders, quotations, invoices, customer management |
| `procurement` | Purchase orders, suppliers, quotes, goods receiving |
| `manufacturing` | Work orders, shop orders, parts, BOMs |
| `maintenance` | Equipment, service orders, fault reporting |
| `finance` | Supplier invoices, GL, cost centers |
| `inventory` | Parts, stock, warehouse locations |
| `projects` | Project management, time reporting |
| `hr` | Employees, time and attendance |

---

## Contributing

Skills are built by the community. If you work with IFS Cloud, your knowledge of how a module works is exactly what's needed here.

See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines. In short:

1. Fork this repo
2. Create a skill file in `skills/` following the naming convention
3. Keep it generic — no real customer data or company-specific values
4. Open a pull request with a brief description of what the skill covers

Not ready for a PR? You can also share a Gist or link in the [Discussions](https://github.com/knakit/ifs-mcp-skills/discussions) tab.

---

## Related

- [IFS MCP Server](https://github.com/knakit/ifs-mcp) — the server that uses these skills

---

> Skills in this repository are provided by the community for informational purposes. Always test against your own IFS instance before using in production workflows.
