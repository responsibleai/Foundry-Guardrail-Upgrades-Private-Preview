# Intervention points (private preview)

An intervention point is a stage in a model or agent interaction where a guardrail checks content. Each control in a guardrail, including each custom rule, runs at one or more intervention points that you select.

When an agent can call tools, read external content, or trigger downstream processes, risks can enter or leave the interaction at several stages, not only in the user's request and the final response. Checking content at each stage lets you stop a risky action before it runs, and stop untrusted content before the agent acts on it.

## The four intervention points

| Intervention point | Content that's checked | Applies to |
|--------------------|------------------------|------------|
| User input | The request sent from a user to a model or agent, sometimes called the *prompt*. | Models and agents |
| Tool input | The tool call that an agent's model proposes, including the tool name and the arguments and data that the agent sends to the tool. | Agents |
| Tool output | The content that a tool returns to the agent, before the agent adds it to its context or returns it to the user. | Agents |
| Output | The final response sent to the user. | Models and agents |

> [!NOTE]
> If you've used the previous guardrail experience, tool input was called *tool call* and tool output was called *tool response*. The behavior is the same.

## What happens when content is blocked

When a control blocks content, the result depends on the intervention point:

| Intervention point | Result of a block |
|--------------------|-------------------|
| User input | The request isn't sent to the model, and the interaction stops. |
| Tool input | The tool call doesn't run. The agent stops until it receives another user input. |
| Tool output | The agent stops immediately. The tool content isn't added to the agent's context or returned to the user. |
| Output | The response isn't returned to the user. |

Built-in controls and custom rules block content in the same way, for models, prompt agents, and hosted agents.

## Examples

| Intervention point | Built-in control example | Custom rule example |
|--------------------|--------------------------|---------------------|
| User input | Block jailbreak attempts. | Block requests that a scope judge labels `out_of_scope`. |
| Tool input | Block hateful content that the agent sends to a tool. | Block calls to a tool named `transfer_funds`. |
| Tool output | Block indirect prompt injections in tool output. | Block tool output that a confidentiality judge labels `confidential`. |
| Output | Block protected material for text. | Block responses that a financial advice judge scores above 0.7 on a 0-1 scale. |

## Supported tools

Controls at tool input and tool output require moderation support from the tool. Currently, the following tools support moderation:

- Azure AI Search
- Azure Functions
- Bing Custom Search
- Bing Grounding
- Browser Automation
- Fabric Data Agent
- OpenAPI
- SharePoint Grounding

If your agent uses a tool that isn't in this list, controls at tool input and tool output, including custom rules, don't run for that tool.

## Performance considerations

Each intervention point that has controls adds latency:

- Built-in controls add latency that depends on content length, the active controls, and the serving environment.
- Custom rules with only deterministic conditions add minimal latency.
- Custom rules that use an LLM judge add the time it takes the judge's model deployment to respond.

To limit latency, add controls only at the intervention points where you need them, and prefer deterministic conditions over LLM judges when they can express your policy.

## Related content

- [Built-in controls](built-in-controls.md)
- [Custom rules and LLM judges](custom-rules.md)
- [Create and manage guardrails](how-to-create-guardrails.md)
