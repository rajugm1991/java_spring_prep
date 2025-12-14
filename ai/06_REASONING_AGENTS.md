# Reasoning & AI Agents: Complete Guide

## Table of Contents
1. [Chain of Thought Reasoning](#chain-of-thought-reasoning)
2. [Tool Usage in LLMs](#tool-usage-in-llms)
3. [Tree of Thought](#tree-of-thought)
4. [Context Engineering](#context-engineering)
5. [AI Agents Architecture](#ai-agents-architecture)
6. [Model Context Protocol](#model-context-protocol)

---

## Chain of Thought Reasoning

### What is Chain of Thought?

**Chain of Thought (CoT)** is a prompting technique that encourages LLMs to reason step-by-step before answering.

### Problem: Direct Answers Can Be Wrong

**Without CoT:**
```
Q: A store has 15 apples. They sell 8. How many left?
A: 7
```
Model might guess without reasoning.

**With CoT:**
```
Q: A store has 15 apples. They sell 8. How many left?
A: Let's think step by step.
1. Store starts with 15 apples
2. They sell 8 apples
3. Remaining = 15 - 8 = 7 apples
So the answer is 7.
```

### Implementation

```python
class ChainOfThoughtReasoning:
    def __init__(self, llm_client):
        self.llm = llm_client
    
    def solve(self, problem: str):
        """Solve problem using chain of thought"""
        prompt = f"""Solve this problem step by step.

Problem: {problem}

Solution: Let's think step by step.
"""
        response = self.llm.generate(prompt)
        return response

# Usage
reasoner = ChainOfThoughtReasoning(llm)
answer = reasoner.solve("If 3x + 5 = 20, what is x?")
```

### Few-Shot CoT

**Provide examples in prompt:**

```python
def few_shot_cot_prompt(problem):
    examples = """Q: A store has 15 apples. They sell 8. How many left?
A: Let's think step by step.
1. Store starts with 15 apples
2. They sell 8 apples
3. Remaining = 15 - 8 = 7 apples
So the answer is 7.

Q: Sarah has 20 books. She gives away 5 and buys 3 more. How many does she have?
A: Let's think step by step.
1. Sarah starts with 20 books
2. She gives away 5: 20 - 5 = 15 books
3. She buys 3 more: 15 + 3 = 18 books
So the answer is 18.

Q: {problem}
A: Let's think step by step.
"""
    return examples
```

### Zero-Shot CoT

**Just add "Let's think step by step":**

```python
def zero_shot_cot(problem):
    prompt = f"""Q: {problem}
A: Let's think step by step.
"""
    return llm.generate(prompt)
```

### Benefits

1. **Better Accuracy:** 20-30% improvement on math/reasoning tasks
2. **Explainable:** Can see model's reasoning process
3. **Error Detection:** Can identify where reasoning goes wrong

---

## Tool Usage in LLMs

### What are Tools?

**Tools** are external functions LLMs can call to extend capabilities:
- Calculator
- Web search
- Database queries
- API calls
- Code execution

### Function Calling

**OpenAI Function Calling:**

```python
import openai

# Define tools
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    }
                },
                "required": ["location"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Perform mathematical calculations",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Mathematical expression"
                    }
                },
                "required": ["expression"]
            }
        }
    }
]

# LLM decides when to call tools
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "What's the weather in NYC and calculate 15 * 23?"}],
    tools=tools,
    tool_choice="auto"  # Let model decide
)

# Response contains tool calls
for tool_call in response.choices[0].message.tool_calls:
    function_name = tool_call.function.name
    arguments = json.loads(tool_call.function.arguments)
    
    # Execute tool
    if function_name == "get_weather":
        result = get_weather(arguments["location"])
    elif function_name == "calculate":
        result = eval(arguments["expression"])  # In production, use safe eval
    
    # Send result back to LLM
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": str(result)
    })
    
    # Get final response
    final_response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=messages
    )
```

### ReAct Pattern

**Reasoning + Acting:**

```python
class ReActAgent:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = {tool.name: tool for tool in tools}
    
    def run(self, question: str, max_steps: int = 10):
        """Run ReAct loop"""
        history = [f"Question: {question}"]
        
        for step in range(max_steps):
            # Think
            thought = self.llm.generate(
                f"{' '.join(history)}\nThought:"
            )
            history.append(f"Thought: {thought}")
            
            # Decide action
            action = self.llm.generate(
                f"{' '.join(history)}\nAction:"
            )
            
            # Parse action
            if action.startswith("Search"):
                query = extract_query(action)
                result = self.tools["search"].execute(query)
                history.append(f"Action: {action}\nObservation: {result}")
            elif action.startswith("Calculate"):
                expr = extract_expression(action)
                result = self.tools["calculator"].execute(expr)
                history.append(f"Action: {action}\nObservation: {result}")
            elif action.startswith("Final Answer"):
                return extract_answer(action)
        
        return "Could not find answer"
```

### Tool Implementation

```python
class Tool:
    def __init__(self, name: str, description: str, func: callable):
        self.name = name
        self.description = description
        self.func = func
    
    def execute(self, **kwargs):
        return self.func(**kwargs)

# Define tools
def get_weather(location: str) -> str:
    # Call weather API
    return f"Weather in {location}: 72°F, sunny"

def calculate(expression: str) -> float:
    # Safe evaluation
    return eval(expression)

tools = [
    Tool("get_weather", "Get weather for location", get_weather),
    Tool("calculate", "Calculate mathematical expression", calculate)
]
```

---

## Tree of Thought

### What is Tree of Thought?

**Tree of Thought (ToT)** explores multiple reasoning paths in parallel, then selects the best.

### Problem with Chain of Thought

**CoT follows one path:**
```
Problem → Step 1 → Step 2 → Step 3 → Answer
```
If path is wrong, answer is wrong.

### Tree of Thought Solution

**Explore multiple paths:**
```
Problem
├─ Path 1: Step 1a → Step 2a → Step 3a → Answer A
├─ Path 2: Step 1b → Step 2b → Step 3b → Answer B
└─ Path 3: Step 1c → Step 2c → Step 3c → Answer C

Evaluate all paths → Select best
```

### Implementation

```python
class TreeOfThought:
    def __init__(self, llm, num_paths=5, max_depth=3):
        self.llm = llm
        self.num_paths = num_paths
        self.max_depth = max_depth
    
    def solve(self, problem: str):
        """Solve using tree of thought"""
        # Root node
        root = ThoughtNode(problem, depth=0)
        
        # Expand tree
        self.expand_tree(root)
        
        # Evaluate all leaf nodes
        leaf_nodes = self.get_leaf_nodes(root)
        scores = [self.evaluate(node) for node in leaf_nodes]
        
        # Select best
        best_idx = scores.index(max(scores))
        best_path = self.get_path_to_node(leaf_nodes[best_idx])
        
        return best_path
    
    def expand_tree(self, node: ThoughtNode):
        """Expand node by generating multiple next steps"""
        if node.depth >= self.max_depth:
            return
        
        # Generate multiple reasoning steps
        next_steps = self.llm.generate_multiple(
            f"Problem: {node.problem}\nCurrent reasoning: {node.reasoning}\nNext step:",
            num_generations=self.num_paths
        )
        
        # Create child nodes
        for step in next_steps:
            child = ThoughtNode(
                problem=node.problem,
                reasoning=node.reasoning + "\n" + step,
                depth=node.depth + 1,
                parent=node
            )
            node.children.append(child)
            self.expand_tree(child)
    
    def evaluate(self, node: ThoughtNode) -> float:
        """Evaluate quality of reasoning path"""
        # Use LLM to score
        score_prompt = f"""Rate this solution on a scale of 1-10:

Problem: {node.problem}
Solution: {node.reasoning}

Score (1-10):"""
        score = self.llm.generate(score_prompt)
        return float(score)

class ThoughtNode:
    def __init__(self, problem: str, reasoning: str = "", depth: int = 0, parent=None):
        self.problem = problem
        self.reasoning = reasoning
        self.depth = depth
        self.parent = parent
        self.children = []
```

### Benefits

1. **Better Accuracy:** Explores multiple solutions
2. **Robust:** Less likely to get stuck on wrong path
3. **Explainable:** Can see all reasoning paths

---

## Context Engineering

### What is Context Engineering?

**Context Engineering** is the art of crafting prompts and context to get the best results from LLMs.

### Principles

#### 1. Clear Instructions

**Bad:**
```
"Write something about AI"
```

**Good:**
```
"Write a 500-word blog post about the impact of AI on healthcare. 
Target audience: healthcare professionals. 
Tone: professional but accessible. 
Include: current applications, future potential, ethical considerations."
```

#### 2. Few-Shot Examples

```python
def few_shot_prompt(task, examples, input_text):
    prompt = f"""Task: {task}

Examples:
{examples}

Input: {input_text}
Output:"""
    return prompt
```

#### 3. Chain of Thought

```python
def cot_prompt(problem):
    return f"""Solve this problem step by step.

Problem: {problem}

Solution: Let's think step by step.
"""
```

#### 4. Role Playing

```python
def role_playing_prompt(role, task, input_text):
    return f"""You are an expert {role}.

Task: {task}

Input: {input_text}

Response:"""
```

### Context Window Management

**Problem:** LLMs have limited context windows (4K-128K tokens).

**Solution:** Manage context efficiently.

```python
class ContextManager:
    def __init__(self, max_tokens=8000):
        self.max_tokens = max_tokens
        self.messages = []
    
    def add_message(self, role: str, content: str):
        """Add message, truncate if needed"""
        self.messages.append({"role": role, "content": content})
        
        # Check token count
        total_tokens = self.count_tokens(self.messages)
        
        if total_tokens > self.max_tokens:
            # Remove oldest messages (keep system message)
            while total_tokens > self.max_tokens and len(self.messages) > 1:
                removed = self.messages.pop(1)  # Keep index 0 (system)
                total_tokens -= self.count_tokens([removed])
    
    def summarize_context(self):
        """Summarize old context to save tokens"""
        old_messages = self.messages[1:-5]  # Keep recent 5
        summary = self.llm.summarize(old_messages)
        self.messages = [self.messages[0], summary] + self.messages[-5:]
```

### Prompt Templates

```python
class PromptTemplate:
    def __init__(self, template: str):
        self.template = template
    
    def format(self, **kwargs):
        return self.template.format(**kwargs)

# Define templates
qa_template = PromptTemplate("""
Context: {context}

Question: {question}

Answer based on the context:
""")

code_generation_template = PromptTemplate("""
Task: {task}
Language: {language}
Requirements: {requirements}

Code:
""")
```

---

## AI Agents Architecture

### What are AI Agents?

**AI Agents** are autonomous systems that can:
1. Perceive environment
2. Make decisions
3. Take actions
4. Learn from feedback

### Agent Architecture

```
┌─────────────┐
│   User      │
└──────┬──────┘
       │
┌──────▼──────────────────┐
│   Agent Orchestrator    │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Planning Module       │
│   - Break down task     │
│   - Create plan         │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Execution Module      │
│   - Execute steps       │
│   - Use tools           │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Memory Module          │
│   - Short-term memory   │
│   - Long-term memory    │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Reflection Module     │
│   - Evaluate results    │
│   - Update plan         │
└─────────────────────────┘
```

### Implementation

```python
class AIAgent:
    def __init__(self, llm, tools, memory):
        self.llm = llm
        self.tools = tools
        self.memory = memory
        self.plan = []
        self.current_step = 0
    
    def run(self, goal: str):
        """Execute agent task"""
        # 1. Plan
        self.plan = self.create_plan(goal)
        
        # 2. Execute
        for step in self.plan:
            result = self.execute_step(step)
            self.memory.store(step, result)
            
            # 3. Reflect
            if not self.evaluate_result(result):
                # Replan if needed
                self.plan = self.replan(goal, self.memory)
        
        return self.memory.get_final_result()
    
    def create_plan(self, goal: str) -> list:
        """Break down goal into steps"""
        prompt = f"""Goal: {goal}

Break this down into concrete steps:
1.
2.
3.
"""
        response = self.llm.generate(prompt)
        steps = self.parse_steps(response)
        return steps
    
    def execute_step(self, step: str):
        """Execute a single step"""
        # Determine if step requires tool
        if self.requires_tool(step):
            tool_name, args = self.parse_tool_call(step)
            result = self.tools[tool_name].execute(**args)
        else:
            result = self.llm.generate(f"Step: {step}\nResult:")
        
        return result
    
    def evaluate_result(self, result: str) -> bool:
        """Evaluate if result is satisfactory"""
        evaluation = self.llm.generate(
            f"Evaluate this result: {result}\nIs it satisfactory? (yes/no):"
        )
        return "yes" in evaluation.lower()
```

### Agent Types

**1. ReAct Agent:**
```python
# Reasoning + Acting
# Thinks, then acts, observes, repeats
```

**2. Plan-and-Execute Agent:**
```python
# Creates plan first, then executes
```

**3. Reflexion Agent:**
```python
# Reflects on past actions, learns from mistakes
```

**4. Multi-Agent System:**
```python
class MultiAgentSystem:
    def __init__(self, agents):
        self.agents = agents
        self.coordinator = Coordinator()
    
    def solve(self, task):
        # Decompose task
        subtasks = self.coordinator.decompose(task)
        
        # Assign to agents
        assignments = self.coordinator.assign(subtasks, self.agents)
        
        # Execute in parallel
        results = [agent.solve(subtask) for agent, subtask in assignments]
        
        # Combine results
        final_result = self.coordinator.combine(results)
        return final_result
```

---

## Model Context Protocol

### What is MCP?

**Model Context Protocol (MCP)** is a standard for how AI systems exchange context and information.

### Key Concepts

**1. Context Providers:**
```python
class ContextProvider:
    def get_context(self, query: str) -> dict:
        """Provide relevant context for query"""
        return {
            "text": "...",
            "metadata": {...},
            "source": "..."
        }
```

**2. Context Consumers:**
```python
class ContextConsumer:
    def consume_context(self, context: dict):
        """Use context to generate response"""
        prompt = f"Context: {context['text']}\nQuestion: {query}"
        return self.llm.generate(prompt)
```

**3. Context Exchange:**
```python
class MCPClient:
    def __init__(self, server_url):
        self.server = server_url
    
    def request_context(self, query: str):
        """Request context from MCP server"""
        response = requests.post(
            f"{self.server}/context",
            json={"query": query}
        )
        return response.json()
    
    def provide_context(self, context: dict):
        """Provide context to MCP server"""
        requests.post(
            f"{self.server}/context",
            json=context
        )
```

### MCP Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ MCP Protocol
┌──────▼──────────────────┐
│   MCP Server            │
│   - Context Store       │
│   - Context Router      │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Context Providers     │
│   - Vector DB           │
│   - Knowledge Base       │
│   - APIs                 │
└─────────────────────────┘
```

### Implementation

```python
class MCPServer:
    def __init__(self):
        self.context_store = {}
        self.providers = []
    
    def register_provider(self, provider: ContextProvider):
        self.providers.append(provider)
    
    def handle_request(self, query: str):
        """Handle context request"""
        # Get context from all providers
        contexts = []
        for provider in self.providers:
            context = provider.get_context(query)
            contexts.append(context)
        
        # Combine and return
        return self.combine_contexts(contexts)
    
    def store_context(self, context: dict):
        """Store context for future use"""
        key = self.generate_key(context)
        self.context_store[key] = context
```

---

## Key Takeaways

1. **Chain of Thought:** Step-by-step reasoning improves accuracy
2. **Tool Usage:** Extend LLM capabilities with external functions
3. **Tree of Thought:** Explore multiple reasoning paths
4. **Context Engineering:** Craft prompts for best results
5. **AI Agents:** Autonomous systems that plan and execute
6. **MCP:** Standard protocol for context exchange

---

## Next Steps

- [Production AI Systems](07_PRODUCTION_AI.md) - Deploying agents in production
- [Vector Embeddings Guide](01_VECTOR_EMBEDDINGS.md) - Foundation for RAG

---

## References

- [Chain of Thought Paper](https://arxiv.org/abs/2201.11903)
- [ReAct Paper](https://arxiv.org/abs/2210.03629)
- [Tree of Thought Paper](https://arxiv.org/abs/2305.10601)

