# Guardrails overview (private preview)

Guardrails detect and block harmful content and risky actions in the models and agents that you build in Microsoft Foundry. You configure a guardrail once and assign it to the models and agents that need it. Foundry then checks content automatically at runtime, without changes to your application code.

This article explains how guardrails work and what you can do with them in the private preview. No prior experience with Foundry guardrails is required.

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features and behavior might change before general availability.

## Why use guardrails

Models and agents can receive malicious input, generate harmful content, or, in the case of agents, take actions that the user didn't intend. Agents increase this risk because they can call tools, read external content, write to data stores, and trigger downstream processes.

Guardrails help you:

- Block harmful content, such as hate or violence, in user input and model output.
- Detect attacks, such as jailbreak attempts and prompt injections hidden in documents or tool output.
- Prevent agents from taking actions that you don't allow, such as calling a specific tool.
- Enforce policies that are specific to your organization, such as keeping a support agent on topic.

## Key concepts

| Concept | Description |
|---------|-------------|
| **Guardrail** | A named collection of controls. You assign a guardrail to one or more models and agents in a project. |
| **Control** | A single check in a guardrail. Each control defines what to detect, the intervention points to check, and the action to take when it detects a match. |
| **Built-in control** | A control that uses a Microsoft classification model to detect a predefined risk, such as violence or jailbreak attempts. See [Built-in controls](built-in-controls.md). |
| **Custom rule** | A control that you define. It matches content by using conditions that you set, and blocks the content when it matches. See [Custom rules and LLM judges](custom-rules.md). |
| **LLM judge** | A model deployment that assesses content by using an evaluation prompt that you write. A custom rule can use a judge's result in its conditions. |
| **Intervention point** | A stage in an interaction where a guardrail checks content: user input, tool input, tool output, or output. See [Intervention points](intervention-points.md). |

## How a guardrail checks an agent interaction

The following steps show where a guardrail checks content during a typical agent interaction. Models without tools use only the first and last steps.

1. **User input.** A user sends a request. The guardrail checks the request before the model processes it.
1. **Tool input.** The agent's model proposes to call a tool with specific arguments. The guardrail checks the proposed call before the tool runs.
1. **Tool output.** The tool returns content. The guardrail checks the content before the agent uses it.
1. **Output.** The agent generates its final response. The guardrail checks the response before the user receives it.

At each intervention point, every control in the guardrail that's configured for that point runs. If any control blocks the content, the interaction stops at that point.

## What you can do in the new guardrail experience

| Task | Description | More information |
|------|-------------|------------------|
| Create a guardrail | Start from a set of default controls and add the controls you need. | [Create and manage guardrails](how-to-create-guardrails.md) |
| Configure built-in controls | Turn on built-in controls, choose intervention points and actions, and set blocking levels for content harms. | [Built-in controls](built-in-controls.md) |
| Add custom rules | Block content or tool calls that match conditions you define, at any intervention point. | [Custom rules and LLM judges](custom-rules.md) |
| Add LLM judges | Use a model to assess meaning or intent, and use the judge's label or score in a custom rule. | [LLM judges](custom-rules.md#llm-judges) |
| Control network egress | Allow or deny outbound connections from hosted agents. | [Network controls](built-in-controls.md#network-controls) |
| Assign a guardrail | Apply a guardrail to one or more models and agents. | [Assign the guardrail](how-to-create-guardrails.md#step-2-select-agents-and-models) |
| Require an organization baseline | Use Azure Policy to require a baseline when teams create or update guardrails. | [Govern guardrails with Azure Policy](azure-policy.md) |
| Test a guardrail | Send test requests in the playground and see which control blocked the content, and where. | [Test a guardrail](how-to-create-guardrails.md#test-a-guardrail) |
| Edit or delete a guardrail | Change controls and assignments, or remove guardrails you no longer need. | [Edit a guardrail](how-to-create-guardrails.md#edit-a-guardrail) |

## Guardrails for models and agents

You can assign the same guardrail to models and agents. Some controls apply only to agents:

| Intervention point | Models | Agents |
|--------------------|--------|--------|
| User input | ✅ | ✅ |
| Tool input | ❌ | ✅ |
| Tool output | ❌ | ✅ |
| Output | ✅ | ✅ |

If a guardrail contains controls that don't apply to a model, such as a custom rule at tool input, those controls don't run for the model. The other controls in the guardrail still run.

### Which guardrail applies

- **Model deployments** use the **Microsoft.DefaultV2** guardrail unless you assign a different guardrail.
- **Agents** use the guardrail that you assign to them. If you don't assign one, an agent uses the guardrail of its underlying model deployment.
- An agent's guardrail fully replaces its model deployment's guardrail. Controls in the model deployment's guardrail don't run for the agent.

For example, a model deployment's guardrail blocks violence at a high blocking level, and an agent that uses that deployment has a guardrail that blocks violence at a low blocking level. Requests to the agent are checked at the low blocking level.

An administrator can also require a baseline through Azure Policy when an ACS-format guardrail is created or updated in scope. With the **Deny** effect, Automatic enforcement adds a separate baseline that runs alongside that guardrail; Manual enforcement checks that the guardrail satisfies the baseline without adding a separate runtime policy. These definitions don't check every model or agent assignment, so verify that the guardrail you select has met the requirement. See [Govern guardrails with Azure Policy](azure-policy.md).

### Scope

- Guardrails apply to [Foundry Models sold directly by Azure](https://learn.microsoft.com/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure), except prompts and completions processed by audio transcription models.
- Guardrails apply only to agents built in Foundry Agent Service, not to other agents registered in the Foundry control plane.

## Runtime enforcement

Foundry enforces guardrails automatically when your application calls a model deployment or an agent. You don't need to change your application code to use built-in controls or custom rules.

When a control blocks content, the API returns an error or a filtered response instead of the blocked content. Your application should handle these responses, for example by showing the user a helpful message.

## Join the private preview

To join, complete the [private preview invitation form](https://aka.ms/agentcontrolssignups). The Microsoft engineering team contacts you and sets up the preview for you.

### What changes after you join

| Task | Behavior in the private preview |
|------|---------------------------------|
| Create a guardrail | Always uses the new experience, which includes custom rules. |
| Edit a guardrail created before you joined | Opens in the previous experience. You can change its built-in controls and assignments, but you can't add custom rules. |
| Use **Microsoft.DefaultV2** | Unchanged. It remains the default guardrail for model deployments and can't be edited. |

To add custom rules for a model or agent that uses an older guardrail, create a new guardrail and assign it to the model or agent.

Spotlighting and Groundedness aren't available in the new experience. They continue to work in guardrails that were created in the previous experience.

## Next step

> [!div class="nextstepaction"]
> [Learn about intervention points](intervention-points.md)
