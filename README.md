# AI Code Assistant: Building Cursor, Windsurf, and Antigravity

A production-grade implementation showing how AI coding assistants like Cursor, Windsurf, and Antigravity work internally using RAG (Retrieval-Augmented Generation).

## Architecture Overview

This project demonstrates the four critical components of a code assistant:

### 1. Code Parsing (AST-Based)
Unlike simple text splitters that break code arbitrarily, this uses Tree-sitter to parse code into an Abstract Syntax Tree (AST). This ensures functions and classes remain intact.

### 2. Vector Storage
Uses ChromaDB with OpenAI embeddings to enable semantic search. The system understands that `get_users()` and "getting the user list" mean the same thing.

### 3. Repository Mapping
For large codebases (100k+ lines), creates a compressed tree structure showing file hierarchy and class/function definitions. This gives the AI a global view.

### 4. RAG Chain
Combines retrieved code context with LLM prompts designed for senior software engineers, ensuring accurate and context-aware responses.

## Key Features

- AST-aware code parsing that preserves semantic integrity
- MMR (Maximal Marginal Relevance) search for diverse, relevant results
- Repository structure mapping for global codebase understanding
- Interactive Q&A mode
- Source document tracking
- Support for multiple programming languages (Python, JavaScript, TypeScript, Java, Go, Rust, C++)

## Installation

```bash
# Install dependencies
pip install -r requirements.txt

# Set your OpenAI API key
export OPENAI_API_KEY='your-api-key-here'
```

## Usage

### Interactive Mode
```bash
python main.py --repo ./your_project --interactive
```

### Single Query
```bash
python main.py --repo ./your_project --query "How does the authentication system work?"
```

### Show Source Documents
```bash
python main.py --repo ./your_project --query "Explain the payment processor" --show-sources
```

### Multiple File Types
```bash
python main.py --repo ./your_project --extensions .py .js .ts --interactive
```

### View Repository Structure
```bash
python main.py --repo ./your_project --show-structure
```

### Force Reindex
```bash
python main.py --repo ./your_project --reindex --interactive
```

## How It Works

### 1. Code Parsing with Tree-sitter
```python
# Instead of breaking code at arbitrary character limits,
# Tree-sitter parses the AST and splits by logical units
loader = GenericLoader.from_filesystem(
    "./your_repo",
    parser=LanguageParser(language=Language.PYTHON, parser_threshold=500)
)
```

### 2. Semantic Vector Search
```python
# Uses MMR (Maximal Marginal Relevance) to avoid returning
# 5 identical code blocks, instead providing diverse relevant results
retriever = db.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 8}
)
```

### 3. Repository Mapping
```
src/
  auth/
    login.py:
      - class AuthManager
      - def login(user, pass)
  db/
    models.py:
      - class User
```

### 4. Context-Aware Prompting
The system uses prompts designed for senior engineers, instructing the AI to:
- Reference actual class and variable names from context
- Admit when context is insufficient
- Use repository maps to suggest relevant files not in search results

## Architecture Diagram

```
User Query
    |
    v
[Repository Mapper] -> Creates global structure view
    |
    v
[Code Parser] -> Parses code via AST (Tree-sitter)
    |
    v
[Vector Store] -> Stores embeddings in ChromaDB
    |
    v
[Retriever] -> MMR search for relevant chunks
    |
    v
[RAG Chain] -> Combines context + repo map + query
    |
    v
[LLM] -> Generates context-aware response
    |
    v
Answer
```

## Why This Approach?

### Problem with Simple Text Splitting
```python
# BAD: This breaks functions mid-way
def process_payment(user, amount):
    if user.balance < amount:  # <- Split happens here
        return False
```

### Solution: AST-Based Splitting
```python
# GOOD: Complete logical units
def process_payment(user, amount):
    if user.balance < amount:
        return False
    user.balance -= amount
    return True
```

## Advanced Features

### MMR vs Simple Similarity Search
- Simple similarity might return 5 nearly identical functions
- MMR ensures diverse but relevant results
- Gives the AI broader codebase perspective

### Repository Mapping Benefits
- AI can suggest files not in search results
- Understands overall architecture
- Essential for codebases over 100k lines

## Example Session

```
$ python main.py --repo ./my_project --interactive

INTERACTIVE CODE ASSISTANT
Ask questions about your codebase. Type 'exit' to stop.

You: How do I refactor the PaymentProcessor to use AsyncAPI?

## License

Proprietary — all rights reserved. This code is published for viewing and evaluation only; no use, copying, modification, redistribution, commercial use, or use as AI/ML training data without written permission. See [LICENSE](LICENSE). Commercial licensing: aianytime07@gmail.com · sonu@aianytime.net.
