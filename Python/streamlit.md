# Streamlit GUI Development for RAG Systems

## 1. Execution Model & Core Concepts

 **Top-to-Bottom Rerun:** Streamlit re-executes the entire Python script from top to bottom on every user interaction (e.g., button clicks, text inputs).

 **CLI Commands:** Run Streamlit apps using `streamlit run <script.py>` instead of `python <script.py>`.

**Live Reload:** Changes saved in the IDE trigger automatic browser re-renders ("Always rerun").

## 2. UI Elements & Layout Syntax

- **`st.title()` & `st.caption()`:** Used for major headers and secondary metadata/subtitles.
- **`st.chat_input()`:** Fixed bottom text area designed for chat interfaces. Returns a string on submit, otherwise `None`.
- **`st.chat_message(role)`:** Acts as a context manager (container) for chat bubbles (`"user"`, `"assistant"`). Must be used with a `with` block:
- 
```python
  with st.chat_message("user"):
	   st.write("Hello")
```
## 3. State Management & Caching

**`st.session_state`:** A key-value dictionary that persists data across script reruns for the duration of a user session. Essential for chat history persistence.

```python
if "messages" not in st.session_state:
    st.session_state.messages = []
```

**`@st.cache_resource`:** Caches global, heavy, or non-serializable objects (e.g., database connections, LLM chains, embedding models) so they are initialized only once across the app life cycle.

```python
@st.cache_resource
def load_chain():
    return conversation_history()
```