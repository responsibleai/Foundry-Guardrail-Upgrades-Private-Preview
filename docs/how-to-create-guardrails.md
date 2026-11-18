# Create and manage guardrails (private preview)

In this article, you use the Microsoft Foundry portal to create a guardrail with built-in controls and custom rules, assign it to models and agents, test it, and manage it.

For background, see [Guardrails overview](overview.md).

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features and behavior might change before general availability.

## Prerequisites

- Access to the guardrails private preview. To request access, see [Join the private preview](overview.md#join-the-private-preview).
- A [Microsoft Foundry project](https://learn.microsoft.com/azure/foundry/how-to/create-projects).
- At least one model deployment in your project.
- The **Foundry Account Owner** role or higher on the Foundry resource.
- To use an LLM judge, a model deployment and permission for the Foundry account's system-assigned managed identity to call it. Your own access to the deployment doesn't give the judge access.
- To test controls at tool input and tool output, an agent that uses at least one [supported tool](intervention-points.md#supported-tools).

## Create a guardrail

1. Go to [Foundry](https://ai.azure.com) and open your project.
1. Select **Build** in the upper-right menu.
1. Select **Guardrails** in the left navigation.
1. Select **Create guardrail**.

The guardrail creation page has three steps: **Configure controls**, **Select agents and models**, and **Review**.

## Step 1: Configure controls

The **Configure controls** step has two sections:

- **Custom**: Custom rules that you define. This section is empty when you start.
- **Built-in**: Built-in controls grouped by risk, such as **Jailbreak**, **Network**, and **Indirect prompt injections**. Some controls are turned on by default.

### Add a custom rule

1. In the **Custom** section, under **Custom rules**, select **Add rule**. The **Add a custom rule** pane opens.
1. In **Rule name**, enter a descriptive name.
1. Next to **If**, define the first condition:
   1. Select a field, such as **Tool name**, or an LLM judge that you added. For the fields available at each intervention point, see [Fields](custom-rules.md#fields).
   1. Select an operator, such as **Equals**.
   1. Enter a value.
1. Optional: To add another condition, select **Add condition**, select **AND** or **OR**, and define the condition. To remove a condition, select **X** next to it.
1. In **Intervention points**, select one or more of **User input**, **Tool input**, **Tool output**, and **Output**.
1. In **Action**, keep **Block**.
1. Select **Add rule**.

If a condition uses a field that isn't available at a selected intervention point, the editor shows a message, and **Add rule** stays unavailable. Change the field or the intervention points to continue.

For details about fields, operators, and combining conditions, see [Custom rules and LLM judges](custom-rules.md#conditions).

### Add an LLM judge

Use an LLM judge when a rule depends on meaning or intent rather than exact values. You add a judge from a condition in the **Add a custom rule** pane.

Before adding the judge, check its model access in the Azure portal:

1. On the Foundry account that owns the guardrail, open **Identity > System assigned**. Confirm that the identity is enabled and note its **Object (principal) ID**. Use the account identity, not the project or agent identity.
1. On the Azure resource that hosts the judge model deployment, open **Access control (IAM)**. Check whether that principal has **Cognitive Services OpenAI User** or equivalent permission to call the model. If it doesn't, ask someone who can assign roles on that resource to grant **Cognitive Services OpenAI User** to the account identity at that resource's scope. Check this even if the deployment and guardrail are in the same account.

1. In the **Add a custom rule** pane, open the field list in a condition. The list shows **Rule fields** and, below them, **LLM judges** that you already added to this guardrail.
1. Select **Add LLM judge**. The **Add LLM judge** pane opens.
1. In **Judge name**, enter a descriptive name. This name appears as a field when you build conditions.
1. In **Model deployment**, select the model deployment that evaluates the content.
1. In **Evaluation prompt**, describe what to assess and define the labels or score scale to return. For guidance, see [Write an effective evaluation prompt](custom-rules.md#write-an-effective-evaluation-prompt).
1. In **Output type**, select **Label** or **Score**.
1. Select **Add LLM judge**.
1. In the condition, select the judge under **LLM judges** in the field list, and then select an operator and a value:
   - For a **Label** judge, select **Equals** or **Does not equal**, and enter a label.
   - For a **Score** judge, select a comparison, such as **Greater than**, and enter a value from **0 to 1** with up to two decimal places. Use that scale in the evaluation prompt.
1. Finish the rule as described in [Add a custom rule](#add-a-custom-rule): select intervention points, keep **Block**, and select **Add rule**.

You can use the same judge in other rules in this guardrail. Judges aren't shared across guardrails.

### Configure built-in controls

1. In the **Built-in** section, expand a risk group.
1. Select the check box next to a risk type to turn it on, or clear it to turn it off. Controls that can't be removed have unavailable check boxes.
1. Review the **Intervention points** for the control.
1. In **Action**, select the action to take.
1. For content harm controls, select a blocking level. For guidance, see [Blocking levels for content harms](built-in-controls.md#blocking-levels-for-content-harms).

To configure network egress rules for hosted agents, select **Manage** next to **Egress rules** in the **Network** section. For details, see [Network controls](built-in-controls.md#network-controls).

When you finish configuring controls, select **Next**.

## Step 2: Select agents and models

1. Select **Add agents**, **Add models**, or both.
1. Select the agents and models to assign the guardrail to. To remove an assignment, clear its selection. A model or agent that you remove reverts to its default guardrail.
1. Select **Save**, and then select **Next**.

The selections take effect after you finish creating the guardrail and the services pick up the assignment. Assign new guardrails to non-production models and agents first, and test the behavior after saving.

## Step 3: Review

1. Review the custom rules, built-in controls, and assignments.
1. Enter a name for the guardrail, or keep the generated name.
1. Select **Create**. The guardrail appears on the **Guardrails** page.

## Assign a guardrail from a model or agent

You can also assign an existing guardrail from a model or agent:

1. Select **Build** in the upper-right menu.
1. Select **Agents** or **Models** in the left navigation, and then select the agent or model.
1. In the left pane of the playground, find the **Guardrails** section, and select **Manage**.
1. Select **Assign a new guardrail**.
1. Select a guardrail, and then select **Assign**. Allow time for the assignment to propagate, and test it before relying on the change.

## Test a guardrail

1. On the **Guardrails** page, select the guardrail's row.
1. In the details pane, select **Try in Playground**. If the button doesn't appear, assign the guardrail to a model or agent first.
1. Send requests that should be blocked and requests that should be allowed for each control and custom rule. To test a rule at tool input or tool output, send a request that causes the agent to call the relevant tool.
1. When a control blocks content, a message in the chat shows which risk was detected and at which intervention point.

Test each LLM judge with representative and borderline content, and refine its evaluation prompt until the results meet your requirements.

## Edit a guardrail

1. On the **Guardrails** page, select the guardrail's name, or select its row and then select **Edit**.
1. Add, change, or remove custom rules and built-in controls, and update assignments as needed.
1. Go to the **Review** step, and save your changes.

To delete a custom rule, select the rule in the **Custom rules** list, and then select **Delete**.

Guardrails that were created before you joined the private preview open in the previous experience. You can change their built-in controls and assignments, but you can't add custom rules. To use custom rules for those models and agents, create a new guardrail and assign it to them.

You can't edit **Microsoft.DefaultV2**.

### When Azure Policy applies

An administrator can require a baseline guardrail through Azure Policy. With the **Deny** effect, Automatic enforcement adds the baseline alongside your guardrail, and Manual enforcement checks whether your guardrail already satisfies the baseline before accepting the save. **Audit** checks the requirement without blocking the save or adding the baseline.

If a save fails because of an Azure Policy requirement, review the baseline with your administrator. Don't remove controls just to make the save succeed. See [Govern guardrails with Azure Policy](azure-policy.md) for setup and troubleshooting.

## Delete a guardrail

You can delete only a guardrail that isn't assigned to any model or agent.

Before deleting a guardrail that an administrator uses as an Azure Policy baseline, ask them to update the policy assignment. Re-save and test the guardrails that used the old Automatic baseline before deleting it. An unassigned baseline can still be required by other guardrails.

1. Assign a different guardrail to each model and agent that uses the guardrail, or remove them from the guardrail.
1. On the **Guardrails** page, select the guardrail's row.
1. In the details pane, select **Delete**.

## Related content

- [Custom rules and LLM judges](custom-rules.md)
- [Built-in controls](built-in-controls.md)
- [Limitations and troubleshooting](limitations.md)
- [Govern guardrails with Azure Policy](azure-policy.md)
