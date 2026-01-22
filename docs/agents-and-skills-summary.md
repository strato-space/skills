# Agents and Skills Summary

## One-line definitions
- **Agent**: the control plane that sets goals, policy, tools, memory strategy, routing, and autonomy.
- **Skill**: a task-specific instruction module loaded on demand, optionally bundling scripts/assets.

## Core differences (beyond context/mount point)
- **Lifecycle**: agents persist across a session; skills are activated only when needed.
- **Authority**: agents decide what to do and when; skills describe how to do a specific task.
- **Permissions**: agent config controls tool access; skills inherit those permissions.
- **Scope**: agents orchestrate workflows; skills are scoped to a narrow job.

## Relationship
- Agents can use skills; skills do not become agents.
- A complex skill can contain multi-step procedures, but it still runs under the agent's control.

## Calling agents from skills (optional)
- A skill can invoke an agent **as an external tool** (e.g., via a CLI command) if the runtime permits shell/tool access.
- This is not inherent to skills; it depends on environment permissions and available tooling.

## Practical rule of thumb
- Use an **agent** to define system behavior and orchestration.
- Use a **skill** to standardize a repeatable task or workflow.
