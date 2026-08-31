
## 1. **Runtime Components & State Contracts**

| **Component / Method** | **Input Contract**                                             | **Output Contract**                            | **Critical Rule / Invariant**                                                               |
| ---------------------- | -------------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `ChatOpenAI.invoke`    | `list[BaseMessage]` or `str`                                   | `AIMessage`                                    | `tool_calls` is an empty list if no tool is requested, or a list of tool call dictionaries. |
| `LangGraph Node`       | `state: StateSchema` (`dict`)                                  | `dict` (only the keys to update)               | If a key uses a reducer (e.g., `operator.add`), wrap the value in a list: `[new_item]`.     |
| `tools_condition`      | `state: StateSchema` (`dict`)                                  | `"tools"` or `"__end__"` (or custom node name) | Inspects the last message in `state["messages"]` for the presence of `tool_calls`.          |
| `ToolNode.invoke`      | `state: StateSchema` (`dict`)                                  | `{"messages": list[ToolMessage]}`              | Executes requested tools and returns `ToolMessage` objects matching each `tool_call_id`.    |
| `StateGraph.compile`   | `checkpointer` (optional), `interrupt_before/after` (optional) | `CompiledGraph`                                | Produces a runnable graph instance adhering to the defined state schema.                    |
| `ToolMessage`          | `content: str`, `tool_call_id: str`                            | `ToolMessage` instance                         | `tool_call_id` must match the corresponding `id` from `AIMessage.tool_calls`.               |

## 2. Graph Construction Operators (Builder Contracts)

| **Builder Method**              | **Input Contract**                                                                                         | **Output Contract**      | **Critical Rule / Invariant**                                                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `builder.add_node`              | `node_name: str`, `action: Callable[[State], dict]`                                                        | `None` (mutates builder) | `action` must accept a single `state` argument and return a `dict`. `node_name` must be unique in the graph registry.                      |
| `builder.add_edge`              | `start_key: str \| START`, `end_key: str \| END`                                                           | `None` (mutates builder) | Deterministic unconditional transition. Both ends must be registered node names (`str`) or built-in sentinel constants (`START`/`END`).    |
| `builder.add_conditional_edges` | `source: str \| START`, `path: Callable[[State], str]`, `path_map: dict[str, str] \| list[str]` (optional) | `None` (mutates builder) | `path` reads `state` (read-only) and returns a target node name (`str`) or `END`. Dictionaries map return values to explicit target nodes. |
