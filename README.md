# .NET Software Engineering with AI Coding Agents
This is my basic .NET setup for Coding Agents that I use on a daily basis.

## skills

Workflow:
- grill-me
- tdd
- code-review
- code-simplification
- unslop
- to-spec
- to-tasks

Conventions:
- dotnet-conventions
- python-conventions
- api-endpoint-conventions

## Core Idea and Workflow

1. I usually start with a  `/grill-me` session if I'm unsure about the problem or need to clarify/expand on, after that, I ask the agent to generate a .MD based on what we discussed.
2. Then I break down the plan into smaller tasks, this could be a new skill like to-spec, but I rather still be on the loop (for now).
3. After breaking down the tasks, I implement each task by specifically saying to the agent to use the `/tdd` skill. 
4. Once the task is implemented, I ask the agent to review the code using the `/code-review` skill and provide feedback (usually this part is done automatically by the agent, without me needing to call it explicitly).
5. If I consider the code to be satisfactory, I ask the agent to update the plan, and I move on to the next task starting a new chat using `/new`.

### Unslop
Unslop is there to make it less painful to talk to an AI coding agent (and reading the output).
