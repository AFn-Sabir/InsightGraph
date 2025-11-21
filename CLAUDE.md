# CLAUDE.md - InsightGraph Repository Guide

## Project Overview

**InsightGraph** is an AI-powered graph database query assistant that enables natural language interaction with Neo4j shipping logistics data. The application uses Google's Gemini AI with LangChain to convert user questions into Cypher queries, execute them, and provide human-readable insights.

### Key Features
- Natural language to Cypher query conversion
- Agentic tool-use architecture for multi-step reasoning
- Real-time database schema introspection
- Error handling and query correction
- Streamlit-based web interface for interactive chat

### Project Purpose
This tool is designed for shipping logistics data analysis, allowing users to query complex relationships between customers, shipments, ports, carriers, vessels, and sentiment data without writing Cypher queries manually.

---

## Codebase Structure

```
InsightGraph/
├── graph_agent.py       # Core agent logic and Neo4j tools
├── interface.py         # Streamlit web UI
├── requirements.txt     # Python dependencies
└── .git/               # Git repository
```

### File Descriptions

#### `graph_agent.py` (114 lines)
**Purpose**: Core agentic logic and database interaction layer.

**Key Components**:
- `Neo4jTools` class: Database interaction wrapper
  - `get_schema()`: Retrieves database schema visualization
  - `run_query(query: str)`: Executes Cypher queries
  - Connection management via Neo4j driver

- `create_agent_runner()`: Factory function that:
  - Initializes Neo4j connection with provided credentials
  - Sets up Google Gemini AI (gemini-2.5-pro model)
  - Binds tools to LLM for function calling
  - Returns configured agent runner function

- `agent_runner()`: Main execution loop
  - Manages conversation state with message history
  - Handles tool calls and responses
  - Implements max 5-turn limit for reasoning
  - Returns natural language answers

**System Prompt Details** (lines 52-65):
- Instructs agent to use tools for schema inspection
- Emphasizes exact schema matching to avoid query errors
- Documents expected node types and relationships

#### `interface.py` (76 lines)
**Purpose**: Web-based user interface using Streamlit.

**Key Components**:
- Sidebar credential input (Neo4j URI, username, password, Gemini API key)
- Agent caching via `@st.cache_resource` decorator
- Chat interface with message history
- Error handling for initialization and query execution

**UI Flow**:
1. User provides credentials in sidebar
2. Agent is initialized and cached
3. User enters questions in chat input
4. Agent processes and displays responses
5. Chat history persists across interactions

---

## Technology Stack

### Core Dependencies
```
streamlit          # Web UI framework
neo4j             # Neo4j database driver
langchain         # LLM orchestration framework
langchain-core    # Core LangChain abstractions
langchain-google-genai  # Google Gemini integration
```

### External Services
- **Neo4j Database**: Cloud-hosted graph database (default: neo4j+s://ee558c73.databases.neo4j.io)
- **Google Gemini API**: LLM provider (model: gemini-2.5-pro)

### Python Version
- Minimum Python 3.8+ (required for LangChain)
- Recommended: Python 3.10+

---

## Database Schema

### Node Types
- `Customer`: End users booking shipments
- `Shipment`: Individual shipping transactions
- `Port`: Loading/discharge locations
- `Carrier`: Shipping companies
- `Vessel`: Ships/vehicles
- `Exception`: Shipping exceptions/events
- `Issue`: Problems/concerns
- `SentimentScore`: Sentiment analysis results

### Relationship Types
- `BOOKS`: Customer → Shipment
- `LOADS_AT`: Shipment → Port (loading)
- `DISCHARGES_AT`: Shipment → Port (unloading)
- `CARRIED_BY`: Shipment → Carrier
- `HAS_SENTIMENT`: Entity → SentimentScore
- `HAS_ISSUE`: Entity → Issue

### Schema Access
The agent uses `get_schema()` tool to dynamically retrieve schema information via `CALL db.schema.visualization()`.

---

## Development Workflows

### Running the Application

```bash
# Install dependencies
pip install -r requirements.txt

# Run the Streamlit app
streamlit run interface.py
```

### Local Development Setup

1. **Clone the repository**
   ```bash
   git clone <repo-url>
   cd InsightGraph
   ```

2. **Set up Python environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Prepare credentials**
   - Neo4j URI, username, and password
   - Google Gemini API key

4. **Run the application**
   ```bash
   streamlit run interface.py
   ```

### Git Workflow

**Current Branch**: `claude/claude-md-mi8rkakugtbgjn5c-015ACiLbDgiuPfDVNpUTWQUL`

**Branch Naming Convention**:
- Feature branches: `claude/<description>-<session-id>`
- Always push to designated claude branches

**Recent Commits**:
- `d7a1795`: use langgraph
- `e7d3cfd`: split into files
- `3fd8c9f`: add requirements
- `9429863`: add webapp

**Best Practices**:
- Commit frequently with descriptive messages
- Push to designated branch after completing features
- Use `git push -u origin <branch-name>` for first push

---

## Code Conventions

### Python Style
- **Function naming**: snake_case (e.g., `create_agent_runner`, `run_query`)
- **Class naming**: PascalCase (e.g., `Neo4jTools`)
- **Constants**: Not explicitly used, but follow UPPER_CASE convention
- **Docstrings**: Present for key functions and classes

### Error Handling
- Try-except blocks wrap database operations
- Errors returned as string messages to LLM for self-correction
- Tool execution errors caught and passed as ToolMessage

### Code Organization
- Separation of concerns: agent logic (graph_agent.py) vs UI (interface.py)
- Factory pattern for agent creation
- Tools as class methods for state management

### Agent Design Patterns
- **Tool binding**: LLM.bind_tools() for function calling
- **Manual tool loop**: Explicit iteration over tool calls (max 5 turns)
- **Message-based state**: Maintains conversation history
- **Caching**: Streamlit cache for agent persistence

---

## Testing and Deployment

### Testing Strategy
- **Manual testing**: Primary approach via Streamlit UI
- **Test queries**: Use example questions to verify agent behavior
- **Schema validation**: Ensure `get_schema()` returns expected structure
- **Error scenarios**: Test invalid queries, connection failures

### Example Test Questions
```
"Which customer has the most negative shipments?"
"What are the top 3 issues associated with shipments?"
"Show me all ports where shipments are loaded."
"Which vessels are used by Carrier X?"
```

### Deployment Considerations
- **Environment variables**: Consider using for credentials (not currently implemented)
- **Connection pooling**: Neo4j driver handles connection management
- **Streamlit Cloud**: Can deploy directly with secrets management
- **Docker**: No Dockerfile present, but could be added

---

## AI Assistant Guidelines

### When Modifying Code

1. **Read First**: Always read relevant files before editing
2. **Preserve Structure**: Maintain separation between agent logic and UI
3. **Test Tools**: Verify Neo4j tools work independently before integrating
4. **Update System Prompt**: If schema changes, update lines 52-65 in graph_agent.py
5. **Handle Credentials**: Never hardcode secrets; use environment variables or user input

### Common Modification Scenarios

#### Adding New Tools
1. Add method to `Neo4jTools` class
2. Include in `tools` list in `create_agent_runner()` (line 47)
3. Update system prompt to document new tool
4. Test with manual tool calls before agent integration

#### Changing LLM Provider
1. Replace `langchain-google-genai` import
2. Update LLM initialization (line 50)
3. Verify tool binding compatibility
4. Test tool call format matches expected structure

#### Extending Database Schema
1. Update system prompt with new nodes/relationships (lines 63-64)
2. Test `get_schema()` returns updated information
3. Create example queries for new schema elements

#### Improving Error Handling
- Focus on `agent_runner()` function (lines 78-112)
- Add specific exception types
- Enhance error messages passed to LLM
- Consider retry logic for transient failures

### Code Quality Checklist
- [ ] All database operations wrapped in try-except
- [ ] Credentials never hardcoded
- [ ] System prompt accurately reflects schema
- [ ] Tool functions have descriptive docstrings
- [ ] UI provides clear feedback for errors
- [ ] Agent has reasonable turn limit (currently 5)

### Performance Considerations
- **Agent caching**: `@st.cache_resource` prevents re-initialization
- **Database connection**: Single driver instance per agent
- **Turn limits**: Prevents infinite loops (max_turns=5)
- **Schema caching**: Consider caching schema results to reduce DB calls

### Security Notes
- **Credential handling**: Currently via Streamlit UI (password fields)
- **Query validation**: LLM generates queries, validate or sanitize if needed
- **Connection security**: Uses neo4j+s:// for encrypted connections
- **API keys**: Stored in session, not persisted

---

## Troubleshooting

### Common Issues

**1. "Failed to initialize the agent"**
- Verify Neo4j credentials and connection
- Check Gemini API key validity
- Ensure Neo4j instance is running and accessible

**2. "Error running query"**
- Query syntax errors - agent should self-correct
- Schema mismatch - run `get_schema()` to verify
- Connection timeout - check network/database status

**3. Agent doesn't return answer after multiple turns**
- Max turns reached (5) - consider increasing
- Tool execution failures - check error messages
- Complex query requiring more reasoning steps

**4. Streamlit cache issues**
- Clear cache: Press 'C' then 'Clear cache' in Streamlit
- Restart application if credentials change
- Check `st.cache_resource` decorator

### Debug Tips
- Add print statements in `agent_runner()` to trace execution
- Log tool call arguments and results
- Use Neo4j Browser to test generated Cypher queries
- Check Streamlit console for detailed error traces

---

## Future Enhancements

### Potential Improvements
1. **Add tests**: Unit tests for Neo4jTools, integration tests for agent
2. **Environment config**: Use .env file for default credentials
3. **Query history**: Store and display past queries
4. **Visualization**: Integrate graph visualization for results
5. **Async operations**: Use async Neo4j driver for better performance
6. **Rate limiting**: Add throttling for API calls
7. **Logging**: Structured logging for debugging and monitoring
8. **Docker**: Containerize for easier deployment
9. **CI/CD**: Automated testing and deployment pipeline

### Planned Features (based on git history)
- LangGraph integration (recent commit suggests exploration)
- Enhanced agentic capabilities
- Improved tool orchestration

---

## Contact and Support

For issues, enhancements, or questions:
- Review git commit history for context
- Check LangChain documentation for framework updates
- Consult Neo4j Cypher manual for query syntax
- Reference Google Gemini API docs for model capabilities

**Last Updated**: 2025-11-21
**Repository**: InsightGraph
**Primary Files**: graph_agent.py, interface.py
