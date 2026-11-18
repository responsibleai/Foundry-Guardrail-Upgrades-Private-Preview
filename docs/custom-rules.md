# Custom rules and LLM judges (private preview)

Built-in controls detect common risks, such as violence or jailbreak attempts. Your organization probably also has policies that are specific to your business, for example:

- An agent must never call a payment tool.
- A support agent must only answer questions about your products.
- Responses must not give investment advice.

Custom rules let you enforce policies like these. You define the conditions to check and the intervention points where to check them. When the conditions match, the guardrail blocks the content.

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features and behavior might change before general availability.

## Parts of a custom rule

| Part | Description |
|------|-------------|
| **Rule name** | A descriptive name, for example *Block high-value transfers*. The name helps you identify the rule when you review a guardrail or investigate a blocked request. |
| **Conditions** | One or more conditions that must be true for the rule to match. |
| **Intervention points** | One or more of **User input**, **Tool input**, **Tool output**, and **Output**. |
| **Action** | **Block**. Block is the only action available for custom rules. |

Foundry converts the conditions that you build in the portal into a policy written in [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/), the policy language of Open Policy Agent. You don't need to know Rego to create custom rules. You can't view or edit the generated Rego.

## Conditions

Each condition compares a field to a value:

```text
If <field> <operator> <value>
```

For example, *If Tool name Equals transfer_funds* matches when the agent proposes to call the tool named `transfer_funds`.

### Fields

A field is the information that a condition checks. There are two kinds of fields:

- **Rule fields** describe the content at an intervention point, such as **User input** or **Tool name**.
- **LLM judges** hold the result of an LLM judge. Each LLM judge that you add appears by name under **LLM judges** in the field list.

Each rule field exists only at certain intervention points:

| Rule field | Content | Available at |
|------------|---------|--------------|
| **User input** | The user's request. | User input |
| **Tool name** | The name of the tool that the agent calls. | Tool input, Tool output |
| **Tool argument (Text)** | The tool argument named `text`, which must be a string. | Tool input |
| **Tool argument (Amount)** | The tool argument named `amount`, which must be a number, not a quoted numeric string. | Tool input |
| **Tool output** | The content that a tool returns to the agent. | Tool output |
| **Output** | The final response to the user. | Output |

The tool argument fields check the fixed, top-level keys `text` and `amount`. The editor doesn't let you choose another argument name or a nested path. For example, **Tool argument (Amount)** checks `{"amount": 15000}`, not `{"payment": {"amount": 15000}}`.

Combine a tool argument condition with **Tool name** using **AND** so the rule applies only to the intended tool. If that tool's required argument is missing or has the wrong type, the guardrail blocks the call instead of treating the condition as false. A different tool doesn't need that argument for this **AND** group.

Without a **Tool name** condition in the same **AND** group, the argument check applies to every tool call at **Tool input**. Combining the conditions with **OR** doesn't restrict the argument check to that tool.

If a condition uses a field that isn't available at every selected intervention point, the editor shows a message such as *This field is only available at: User input. Update the intervention points or choose a different field.* You can't add the rule until you change the field or the intervention points.

### Operators

The operators that you can choose depend on the field:

| Field | Operators |
|-------|-----------|
| **User input**, **Tool name**, **Tool argument (Text)**, **Tool output**, **Output** | Equals, Does not equal, Contains, Does not contain, Starts with, Ends with, Matches regex |
| **Tool argument (Amount)** | Greater than, Greater than or equal to, Less than, Less than or equal to, Equals |
| LLM judge with **Label** output | Equals, Does not equal |
| LLM judge with **Score** output | Greater than, Greater than or equal to, Less than, Less than or equal to, Equals |

For example, *If Tool argument (Amount) Greater than 10000* matches a tool call whose amount argument is more than 10,000, and *If Output Matches regex `\b\d{3}-\d{2}-\d{4}\b`* matches a response that contains a number in a US Social Security number format.

Text comparisons are case-sensitive. For example, **Contains** `Project Contoso` doesn't match `project contoso`. **Matches regex** accepts RE2-compatible patterns; lookarounds and backreferences aren't supported. Use `(?i)` in a regex when you need case-insensitive matching.


### Combine conditions

To add a condition, select **Add condition**, and then select **AND** or **OR** to combine it with the previous condition. To remove a condition, select **X** next to it.

Conditions follow standard Boolean logic:

- **AND** matches only when both conditions are true.
- **OR** matches when either condition is true.
- **AND** is evaluated before **OR**. For example, *A OR B AND C* is evaluated as *A OR (B AND C)*.

The editor doesn't support parentheses. To express *(A OR B) AND C*, write *A AND C OR B AND C*.


## LLM judges

Some policies depend on meaning or intent, such as "the request is unrelated to customer support." Exact-match conditions can't express these policies. For these cases, use an LLM judge.

An LLM judge sends content to a model deployment in your project, together with an evaluation prompt that you write. The model returns a label or a score. You then use that result as a field in a custom rule condition.

| Setting | Description |
|---------|-------------|
| **Judge name** | A descriptive name, for example *High-risk request detector*. The name appears as a field when you build conditions. |
| **Model deployment** | The model deployment in your project that evaluates the content. |
| **Evaluation prompt** | Instructions that describe what to assess and define the labels or score scale to return. |
| **Output type** | **Label** or **Score**. |

### Choose an output type

| Output type | Use when | Example condition |
|-------------|----------|-------------------|
| **Label** | Content falls into distinct categories. | *If Scope detector Equals out_of_scope* |
| **Score** | Risk is a matter of degree, and you want to set a threshold. | *If Financial advice detector Greater than 0.7* |

The editor accepts score thresholds from **0 to 1**, with up to two decimal places. Use the same scale in your evaluation prompt.

### Content sent to a judge

You can use a judge at any of the four intervention points. It receives the content for that point:

| Intervention point | Judge input |
|--------------------|-------------|
| User input | The user-input content. |
| Tool input | The proposed tool call's arguments. |
| Tool output | The tool's returned content. |
| Output | The response content. |

Don't assume the judge receives the full conversation or the tool name. Use **Tool name** as a separate condition when the rule needs to distinguish tools.

### Write an effective evaluation prompt

- State exactly what the judge should assess.
- For labels, define a small set of labels that don't overlap, and describe when to use each one.
- For scores, define the scale and what the low and high ends mean.
- Tell the judge to return only the label or score, without explanation.
- Include short examples of content that should and shouldn't match.

Label example:

```text
You review requests sent to a customer support agent for a telecommunications company.
Classify the request with exactly one label:
- in_scope: The request is about the customer's account, billing, data usage, devices, or service.
- out_of_scope: The request is about any other topic.
Return only the label.
```

Score example:

```text
Rate how strongly the response gives personalized investment advice, such as recommending
specific stocks, funds, or allocations for the user.
Use a scale from 0 to 1, where 0 means no investment advice and 1 means explicit,
personalized recommendations to buy or sell specific investments.
Return only the number.
```

### LLM judge considerations

- **Results can vary.** The rule logic is deterministic, but a judge's assessment comes from a language model and can differ for similar content. Test each judge with representative and borderline content before you rely on it.
- **Latency.** Each judge evaluation calls the model deployment, which adds latency at every intervention point where the rule runs. A tool-name condition narrows when the rule matches; it doesn't prevent the judge call.
- **Quota and cost.** Judge evaluations consume the model deployment's quota and are billed as usage of that deployment.
- **Failures block content.** Rules fail closed. If a judge call fails, for example because it times out or is throttled, the rule blocks the content. Make sure the judge's model deployment has enough quota for your traffic.
- **The label `uncertain` is reserved.** Foundry treats it as an incomplete judge result and blocks the content. Don't use it as one of your custom labels.
- **The deployment's guardrail applies.** The guardrail assigned to the judge's model deployment also applies to judge calls. If that guardrail blocks the content sent to the judge, the judge call fails, and the rule blocks the content. Choose a deployment whose guardrail doesn't block the content that the judge needs to evaluate.
- **Reuse within a guardrail.** You can use a judge in several rules in the same guardrail. Judges can't be shared across guardrails. To use the same judge in another guardrail, add it again in that guardrail.

## Choose between rule fields and LLM judges

| Use rule fields when | Use an LLM judge when |
|----------------------|-----------------------|
| The policy can be expressed as exact values, text patterns, or numeric thresholds. | The policy depends on meaning, intent, or context. |
| You need the same result for the same input every time. | Some variation in results is acceptable. |
| Latency must be as low as possible. | The added latency of a model call is acceptable. |

You can combine both in one rule. For example, *If Tool name Equals send_email AND Tone detector Equals hostile* blocks a `send_email` call when the judge labels its arguments `hostile`. The judge still evaluates content at the rule's configured intervention points, even when the tool-name condition doesn't match. If the judge fails, the guardrail blocks the content even for other tools.

## How custom rules work with built-in controls

Custom rules run in addition to the built-in controls in the same guardrail. Content is blocked if any control blocks it, whether that control is a built-in control or a custom rule. Custom rules don't change the configuration of built-in controls.

## Examples

| Goal | Conditions | Intervention points |
|------|------------|---------------------|
| Prevent an agent from moving money | *If Tool name Equals transfer_funds* | Tool input |
| Block large transfers only | *If Tool name Equals transfer_funds AND Tool argument (Amount) Greater than 10000* | Tool input |
| Stop internal code names from reaching users | *If Output Contains Project Contoso* | Output |
| Keep a support agent on topic | *If Scope detector Equals out_of_scope* | User input |
| Stop confidential tool content from reaching the agent | *If Confidentiality detector Equals confidential* | Tool output |
| Stop investment advice in responses | *If Financial advice detector Greater than 0.7* | Output |

## Next step

> [!div class="nextstepaction"]
> [Quickstart: Block a tool call with a custom rule](quickstart-custom-rule.md)
