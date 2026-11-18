# Limitations and troubleshooting (private preview)

This article lists known limitations of the new guardrail experience during the private preview and describes how to resolve common issues.

## Limitations

| Area | Limitation |
|------|------------|
| Access | Available only to customers who complete the private preview invitation form and are set up by the Microsoft engineering team. |
| Custom rule actions | **Block** is the only action. |
| Condition logic | Conditions can be combined with **AND** and **OR** only. Parentheses aren't supported. |
| Unavailable built-in controls | Spotlighting and Groundedness aren't available in the new experience. They continue to work in guardrails created in the previous experience. |
| Older guardrails | Guardrails created before you joined the preview open in the previous experience and can't contain custom rules. |
| Microsoft.DefaultV2 | Can't be edited and doesn't support custom rules. |
| Tool intervention points | Controls at tool input and tool output run only for agents, and only for [supported tools](intervention-points.md#supported-tools). |
| Network controls | Apply only to hosted agents. |
| LLM judges | Results can vary for similar content. Each evaluation adds latency and uses the model deployment's quota. If a judge call fails, the rule blocks the content. Judges can't be shared across guardrails. |
| Judge score thresholds | The editor accepts values from 0 to 1, with up to two decimal places. |
| Tool argument fields | Check only the top-level `text` and `amount` arguments. You can't select another key or a nested path. |
| Azure Policy baselines | Require separate administrator setup and preview support for the target resource and region. See [Govern guardrails with Azure Policy](azure-policy.md). |
| Generated policy | You can't view or edit the Rego policy that Foundry generates from your conditions. |
| Agent support | Guardrails apply only to agents built in Foundry Agent Service. |
| Service level | No service-level agreement. Not recommended for production workloads. |

## Troubleshooting

### Can't add a custom rule

**Symptom:** The editor shows *This field is only available at: ...*, and **Add rule** is unavailable.

**Cause:** A condition uses a field that doesn't exist at one of the selected intervention points. For example, **User input** is available only at user input, and **Tool argument (Amount)** is available only at tool input. For the full list, see [Fields](custom-rules.md#fields).

**Resolution:** Change the intervention points to ones where the field is available, or choose a different field. If you need the rule at several intervention points with different fields, create a separate rule for each.

### A custom rule doesn't block content

**Possible causes:**

- The guardrail isn't assigned to the model or agent that you're testing.
- The rule runs only at tool input or tool output, and you're testing a model rather than an agent.
- The agent uses a tool that doesn't support moderation.
- The condition value doesn't exactly match the content, for example the tool name differs.
- You changed the guardrail or assignment, but the serving service hasn't picked up the change yet.
- An LLM judge returned a different label or score than you expected.
- **AND** and **OR** are combined in a way that's evaluated differently than you intended. **AND** is evaluated before **OR**.

**Resolution:**

- In the playground, confirm the assigned guardrail under **Guardrails**.
- Check the rule's conditions and intervention points.
- Test the LLM judge with the same content, and refine its evaluation prompt or threshold.
- After saving a change, allow time for it to propagate and repeat both allowed and blocked test cases.

### Legitimate content is blocked

**Possible causes:**

- A condition is broader than intended.
- An LLM judge's labels or score threshold are too loose.
- A built-in control blocked the content, for example because its blocking level is set to **High**.
- An LLM judge call failed. Rules fail closed, so a failed judge call blocks the content.
- A tool argument rule applies, but the expected `text` or `amount` argument is missing or has the wrong type. Without **Tool name** in the same **AND** group, this can also block calls to other tools.
- An administrator's Azure Policy baseline adds a required control through Automatic enforcement.

**Resolution:**

- Review the block message in the playground to identify which control blocked the content and at which intervention point.
- Narrow the rule by adding conditions with **AND**, or adjust the judge's threshold.
- Add examples of allowed content to the judge's evaluation prompt.
- Lower the blocking level of the built-in control, and test again.
- If a judge call failed, check the judge's model deployment for throttling or quota limits, and check whether that deployment's guardrail blocks the content sent to the judge.
- If even benign requests are blocked and the judge model reports `401 PermissionDenied` or `Unauthorized`, check the [judge's model access](how-to-create-guardrails.md#add-an-llm-judge). The guardrail blocks the request when the judge can't evaluate it; that doesn't mean the content matched your rule. After a role change propagates, retry one request that should pass and one that should be blocked.
- Check the tool's argument names and types against the [rule fields](custom-rules.md#fields).
- If an Azure Policy baseline applies, ask your administrator to review it. Changing your local guardrail doesn't remove the baseline.

### Can't add custom rules to an existing guardrail

**Cause:** The guardrail was created before you joined the private preview, or it's a Microsoft default guardrail.

**Resolution:** Create a new guardrail with the controls and custom rules that you need, and assign it to your models and agents.

### Requests are slower after you add custom rules

**Cause:** Each LLM judge evaluation calls a model deployment.

**Resolution:** Use LLM judges only at the intervention points where you need them, and use rule fields where possible. Adding a tool-name condition narrows when a rule matches, but doesn't skip the judge call.

## Related content

- [Guardrails overview](overview.md)
- [Custom rules and LLM judges](custom-rules.md)
- [Create and manage guardrails](how-to-create-guardrails.md)
- [Govern guardrails with Azure Policy](azure-policy.md)
