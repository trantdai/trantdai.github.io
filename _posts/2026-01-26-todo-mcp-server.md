---
title: Building a Todo MCP Server - Extending AI Assistants with Custom Tools
layout: post
post-image: "/assets/images/blog/20260126/mcp-todo-server.svg"
description: A Model Context Protocol (MCP) server implementation for todo list management with SQLite persistence
tags:
- project
- mcp-server
- ai-powered-application
- blog
---

# Introduction

This project demonstrates the implementation of a Model Context Protocol (MCP) server that provides todo list management capabilities to AI assistants like Claude Desktop. The Model Context Protocol is an open standard that enables seamless integration between AI applications and external data sources, allowing AI assistants to access and manipulate data through well-defined tools and resources. This Todo MCP server showcases how to build a practical MCP server with persistent storage using SQLite, comprehensive CRUD operations, and proper error handling. By implementing this server, AI assistants gain the ability to create, read, update, and delete todo items, making them more useful for task management workflows.

# What is Model Context Protocol (MCP)?

The Model Context Protocol (MCP) is an open protocol developed by Anthropic that standardizes how AI applications communicate with external systems and data sources. MCP enables:

- **Standardized Integration**: A common protocol for connecting AI assistants to various data sources and tools
- **Tool Exposure**: Servers can expose functions (tools) that AI assistants can invoke
- **Resource Sharing**: Servers can provide access to data resources like files, databases, or APIs
- **Bidirectional Communication**: AI assistants can both query data and perform actions through MCP servers

MCP servers act as bridges between AI assistants and external systems, extending the capabilities of AI beyond their base knowledge and enabling them to interact with real-world data and services.

# High Level Architecture and Design

![Todo MCP Server Architecture](/assets/images/blog/20260126/todo_mcp_architecture.svg "Todo MCP Server Architecture")

The Todo MCP server is built upon the following components:

- **MCP Server Implementation**
  - [Server code](https://github.com/trantdai/genai/blob/main/mcp/todo-mcp-server/src/todo_mcp_server/server.py)
  - [Database models](https://github.com/trantdai/genai/blob/main/mcp/todo-mcp-server/src/todo_mcp_server/models.py)
  - [Database operations](https://github.com/trantdai/genai/blob/main/mcp/todo-mcp-server/src/todo_mcp_server/database.py)

- **Core Features**
  - SQLite database for persistent storage
  - CRUD operations for todo items
  - Status management (pending, in-progress, completed)
  - Priority levels (low, medium, high)
  - Search and filtering capabilities
  - Comprehensive error handling

- **MCP Protocol Components**
  - Tools: Functions that AI assistants can invoke
  - Resources: Data endpoints that provide todo information
  - Prompts: Pre-defined interaction patterns

# Server Implementation

## Database Schema

The Todo MCP server uses SQLite with the following schema:

```python
class Todo(Base):
    __tablename__ = "todos"

    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String, nullable=False)
    description = Column(String)
    status = Column(String, default="pending")  # pending, in-progress, completed
    priority = Column(String, default="medium")  # low, medium, high
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

## Available Tools

The server exposes the following tools to AI assistants:

### 1. create_todo
Creates a new todo item with title, description, status, and priority.

```python
{
    "name": "create_todo",
    "description": "Create a new todo item",
    "inputSchema": {
        "type": "object",
        "properties": {
            "title": {"type": "string"},
            "description": {"type": "string"},
            "status": {"type": "string", "enum": ["pending", "in-progress", "completed"]},
            "priority": {"type": "string", "enum": ["low", "medium", "high"]}
        },
        "required": ["title"]
    }
}
```

### 2. list_todos
Retrieves all todo items with optional filtering by status and priority.

```python
{
    "name": "list_todos",
    "description": "List all todo items with optional filtering",
    "inputSchema": {
        "type": "object",
        "properties": {
            "status": {"type": "string", "enum": ["pending", "in-progress", "completed"]},
            "priority": {"type": "string", "enum": ["low", "medium", "high"]}
        }
    }
}
```

### 3. get_todo
Retrieves a specific todo item by ID.

### 4. update_todo
Updates an existing todo item's properties.

### 5. delete_todo
Deletes a todo item by ID.

### 6. search_todos
Searches todo items by keyword in title or description.

## Resources

The server provides a resource endpoint for accessing todo data:

```
todo://all
```

This resource returns all todos in a structured format that AI assistants can read and understand.

# Installation and Setup

## Prerequisites

- Python 3.10 or higher
- pip package manager
- Claude Desktop (for testing with AI assistant)

## Installation Steps

1. Clone the repository:
```bash
git clone https://github.com/trantdai/genai.git
cd genai/mcp/todo-mcp-server
```

2. Install the package:
```bash
pip install -e .
```

3. Configure Claude Desktop to use the MCP server by editing the configuration file:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`

**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Add the following configuration:

```json
{
  "mcpServers": {
    "todo": {
      "command": "python",
      "args": ["-m", "todo_mcp_server"],
      "env": {
        "TODO_DB_PATH": "/path/to/your/todos.db"
      }
    }
  }
}
```

4. Restart Claude Desktop to load the MCP server.

# Usage Examples

## Creating Todos

Once connected to Claude Desktop, you can interact with the todo server naturally:

**User**: "Create a todo to review the MCP documentation with high priority"

**Claude**: *Uses create_todo tool*

```json
{
  "title": "Review MCP documentation",
  "description": "Read through the Model Context Protocol documentation",
  "priority": "high",
  "status": "pending"
}
```

## Listing and Filtering Todos

**User**: "Show me all high priority todos"

**Claude**: *Uses list_todos tool with priority filter*

![List Todos Example](/assets/images/blog/20260126/list_todos_example.svg "List Todos Example")

## Updating Todo Status

**User**: "Mark todo #1 as in-progress"

**Claude**: *Uses update_todo tool*

```json
{
  "id": 1,
  "status": "in-progress"
}
```

## Searching Todos

**User**: "Find all todos related to documentation"

**Claude**: *Uses search_todos tool*

![Search Todos Example](/assets/images/blog/20260126/search_todos_example.svg "Search Todos Example")

# Development Workflow

## Project Structure

```
todo-mcp-server/
├── src/
│   └── todo_mcp_server/
│       ├── __init__.py
│       ├── server.py          # Main MCP server implementation
│       ├── models.py           # SQLAlchemy models
│       └── database.py         # Database operations
├── docs/
│   ├── README.md              # Main documentation
│   ├── ARCHITECTURE.md        # Architecture details
│   ├── API.md                 # API reference
│   └── DEVELOPMENT.md         # Development guide
├── tests/                     # Test suite
├── pyproject.toml            # Project configuration
└── README.md
```

## Testing

The server includes comprehensive tests for all functionality:

```bash
# Run tests
pytest tests/

# Run with coverage
pytest --cov=todo_mcp_server tests/
```

## Development Mode

For development, install with development dependencies:

```bash
pip install -e ".[dev]"
```

This includes:
- pytest for testing
- black for code formatting
- mypy for type checking
- ruff for linting

# Key Features and Benefits

## 1. Persistent Storage
- SQLite database ensures todos persist across sessions
- Automatic database initialization
- Transaction management for data integrity

## 2. Comprehensive CRUD Operations
- Full create, read, update, delete functionality
- Batch operations support
- Flexible filtering and searching

## 3. Status and Priority Management
- Three status levels: pending, in-progress, completed
- Three priority levels: low, medium, high
- Easy status transitions

## 4. Error Handling
- Graceful error handling for all operations
- Informative error messages
- Database connection management

## 5. MCP Protocol Compliance
- Follows MCP specification
- Proper tool and resource definitions
- JSON-RPC 2.0 communication

## 6. Extensibility
- Clean architecture for adding new features
- Modular design
- Well-documented codebase

# Integration with AI Assistants

The Todo MCP server seamlessly integrates with Claude Desktop, enabling natural language interactions for todo management:

1. **Natural Language Commands**: Users can create, update, and manage todos using conversational language
2. **Context Awareness**: Claude can understand complex requests and break them down into appropriate tool calls
3. **Intelligent Filtering**: Claude can interpret user intent and apply appropriate filters
4. **Batch Operations**: Multiple todos can be managed in a single conversation

![Claude Desktop Integration](/assets/images/blog/20260126/claude_desktop_integration.svg "Claude Desktop Integration")

# Future Enhancements

Potential improvements for the Todo MCP server:

1. **Due Dates and Reminders**: Add temporal features for deadline management
2. **Tags and Categories**: Implement tagging system for better organization
3. **Subtasks**: Support hierarchical todo structures
4. **Collaboration**: Multi-user support with sharing capabilities
5. **Attachments**: Allow file attachments to todos
6. **Recurring Todos**: Support for repeating tasks
7. **Export/Import**: Data portability features
8. **Analytics**: Usage statistics and productivity insights

# Lessons Learned

## MCP Server Development

1. **Protocol Understanding**: Deep understanding of MCP specification is crucial
2. **Error Handling**: Robust error handling improves user experience significantly
3. **Database Design**: Simple, normalized schema works best for MCP servers
4. **Tool Design**: Tools should be atomic and focused on single responsibilities
5. **Documentation**: Comprehensive documentation is essential for adoption

## Best Practices

1. **Type Safety**: Use type hints throughout the codebase
2. **Testing**: Comprehensive test coverage ensures reliability
3. **Logging**: Proper logging aids debugging and monitoring
4. **Configuration**: Environment-based configuration provides flexibility
5. **Versioning**: Semantic versioning helps manage compatibility

# Conclusion

The Todo MCP server demonstrates how to build a practical, production-ready MCP server that extends AI assistant capabilities. By following the Model Context Protocol specification and implementing robust database operations, this server provides a solid foundation for task management through AI assistants. The project showcases best practices in MCP server development, including proper error handling, comprehensive testing, and clear documentation.

This implementation can serve as a template for building other MCP servers that connect AI assistants to various data sources and services, enabling more powerful and useful AI-powered workflows.

# References

1. [Todo MCP Server GitHub Repository](https://github.com/trantdai/genai/tree/main/mcp/todo-mcp-server)
2. [Model Context Protocol Specification](https://modelcontextprotocol.io/)
3. [MCP Python SDK Documentation](https://github.com/modelcontextprotocol/python-sdk)
4. [Claude Desktop Documentation](https://claude.ai/desktop)
5. [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
6. [Building MCP Servers Guide](https://modelcontextprotocol.io/docs/building-servers)
