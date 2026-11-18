# Built-in controls (private preview)

Built-in controls detect predefined safety and security risks by using classification models from [Azure AI Content Safety](https://learn.microsoft.com/azure/ai-services/content-safety/overview). You turn them on and configure them in the **Built-in** section of the guardrail creation page.

For risks that are specific to your business, use [custom rules](custom-rules.md) together with built-in controls.

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features and behavior might change before general availability.

## Available built-in risks

| Risk | What it detects | Models | Agents |
|------|-----------------|--------|--------|
| [Hate](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-severity-levels#harm-category-descriptions) | Content that attacks or discriminates against people based on attributes such as race, ethnicity, religion, gender, or disability. | ✅ | ✅ |
| [Sexual](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-severity-levels#harm-category-descriptions) | Sexually explicit or abusive content. | ✅ | ✅ |
| [Self-harm](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-severity-levels#harm-category-descriptions) | Content that describes or encourages harming oneself. | ✅ | ✅ |
| [Violence](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-severity-levels#harm-category-descriptions) | Content that describes, promotes, or glorifies violence. | ✅ | ✅ |
| [Jailbreak](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-prompt-shields) | Attempts in user input to make the model ignore its instructions or safety rules. Also called *user prompt attacks*. Checked only at user input. | ✅ | ✅ |
| [Indirect prompt injections](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-prompt-shields) | Malicious instructions hidden in content from external sources, such as documents or tool output. Also called *indirect attacks*. | ✅ | ✅ |
| [Protected material for text](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-protected-material) | Known copyrighted text, such as song lyrics, articles, and recipes, in generated content. | ✅ | ✅ |
| [Protected material for code](https://learn.microsoft.com/azure/ai-services/content-safety/quickstart-protected-material-code) | Generated code that matches code in public GitHub repositories. | ✅ | ✅ |
| [Personally identifiable information (PII) (preview)](https://learn.microsoft.com/azure/foundry/openai/concepts/content-filter-personal-information) | Personal information, such as email addresses, phone numbers, and government identification numbers. | ✅ | ✅ |
| [Task Adherence (preview)](https://learn.microsoft.com/azure/foundry/guardrails/task-adherence) | Agent tool calls that don't match the user's intent, such as deleting data when the user asked only to view it. | ✅ | ✅ |
| [Network egress (preview)](#network-controls) | Outbound connections from hosted agents to destinations that you don't allow. | ❌ | Hosted agents only |

For each risk, the guardrail creation page shows only the intervention points and actions that apply to it. For example, jailbreak attempts are, by definition, part of user input, so the Jailbreak control runs only at user input.

Spotlighting and Groundedness aren't available in the new guardrail experience. They continue to work in guardrails that were created in the previous experience.


## Blocking levels for content harms

Controls for Hate, Sexual, Self-harm, and Violence use a blocking level. The classifier assigns each piece of content a severity of safe, low, medium, or high. The blocking level determines which severities are blocked:

| Blocking level | Blocks | Notes |
|----------------|--------|-------|
| **Off** | Nothing. | You can turn off content harm controls for user input. Turning them off for output requires approval for [modified guardrails](https://ncv.microsoft.com/uEfCgnITdR). |
| **Low** | High severity only. | Least restrictive. |
| **Medium** | Medium and high severity. | |
| **High** | Low, medium, and high severity. | Most restrictive. |

Choose a blocking level by weighing the cost of allowing harmful content against the cost of blocking legitimate content. Test with content that represents your workload before you use the guardrail in production.

## Default controls

When you create a guardrail, some controls are added by default. Some default controls, such as content harm controls at user input and output, can't be removed. You can change their configuration, such as the blocking level.

New model deployments use the **Microsoft.DefaultV2** guardrail until you assign a different guardrail. For the controls in Microsoft.DefaultV2, see [Default guardrail policies](https://learn.microsoft.com/azure/foundry/guardrails/default-safety-policies). You can't edit Microsoft.DefaultV2.

## Actions

When a built-in control detects a risk, it takes the action that you select for it. The **Action** column for each control lists the actions available for that risk.

| Action | Models | Agents |
|--------|--------|--------|
| **Block** | ✅ | ✅ |
| Annotate only (report the detection without blocking) | ✅ | ❌ |

For the four content harm risks, annotate-only on output requires approval for modified guardrails.

## Network controls

Network controls govern the outbound connections that a hosted agent makes, so the agent reaches only the destinations you allow. You manage them in the **Network** section of the guardrail creation page by configuring **Egress rules** for **Outbound requests**. The action is **Deny**.

Network controls apply only to hosted agents. Prompt agents and models aren't affected by these settings. For details, see [Add guardrails to a hosted agent](https://learn.microsoft.com/azure/foundry/agents/how-to/add-hosted-agent-guardrails#network-egress-controls-preview).

## Related content

- [Custom rules and LLM judges](custom-rules.md)
- [Intervention points](intervention-points.md)
- [Create and manage guardrails](how-to-create-guardrails.md)
