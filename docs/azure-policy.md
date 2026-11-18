# Govern guardrails with Azure Policy (private preview)

Use Azure Policy to require an organization baseline when teams create or update Foundry guardrails. An administrator creates the baseline guardrail, then assigns an Azure Policy definition that references it. Teams can keep their own controls while meeting that requirement.

For example, an administrator can require content harm controls across a test resource group. A team can add a custom rule that keeps its support agent on topic without removing the required controls.

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features and behavior might change before general availability.

## Prerequisites

- Access to the guardrails private preview. See [Join the private preview](overview.md#join-the-private-preview).
- Confirmation from your Microsoft contact that Azure Policy baseline enforcement is enabled for your target Foundry resource and region. A definition appearing in Azure Policy doesn't confirm regional support.
- A test Foundry resource and project where you can create guardrails.
- Permission to create the baseline guardrail and manage the Azure Policy assignment. **Resource Policy Contributor** includes policy-assignment permissions but doesn't grant permission to create or edit Foundry guardrails.
- A non-production model deployment or agent for runtime testing.

These definitions apply to guardrails in Agent Control Specification (ACS) format, which the new guardrail experience creates. They don't convert older guardrails or govern agents outside Foundry Agent Service.

They evaluate guardrail resources, not every model or agent assignment. Selecting a default guardrail, an older guardrail, or one saved before the requirement applied doesn't by itself enforce the baseline.

## Choose how to enforce the baseline

The two built-in definitions use the same baseline reference but enforce it differently:

| Definition | With effect **Deny** | Use when |
|------------|----------------------|----------|
| **[Preview]: Microsoft Foundry Guardrails should automatically enforce an ACS policy baseline** | Foundry enforces the baseline alongside the team's guardrail. The team's authored rules stay separate. | You want the administrator's baseline to run without requiring each team to reproduce its rules. |
| **[Preview]: Microsoft Foundry Guardrails should satisfy an ACS policy baseline without automatic changes** | Foundry checks that the team's guardrail already satisfies the baseline and rejects a save that doesn't pass. It doesn't add the baseline as a separate runtime policy. | Teams need to author their own guardrails while preserving the baseline's blocking behavior. |

The first definition uses **Automatic** reconciliation. The second uses **Manual** reconciliation. You choose the behavior by choosing the definition, not by adding an assignment parameter.

For Automatic enforcement, both policies apply at their configured intervention points. If either policy blocks content, the request is blocked. Saving a more permissive local rule doesn't remove the required baseline.

For Manual enforcement, Foundry can compare the guardrail's blocking behavior against the baseline. Matching rule names isn't enough. A custom rule or LLM judge must preserve the baseline's required behavior. An unsupported comparison or a timeout isn't a successful check and can prevent the save.

Supported stricter thresholds and additional custom rules can pass this comparison. The inputs must also be compatible: changing the enabled classifiers, tool configuration, or references to blocklists and other moderation resources can prevent a comparison. Keep each required LLM judge's evaluation prompt, including whitespace, model deployment, and intervention points unchanged. A different judge doesn't count as the baseline judge, even if its prompt sounds similar.

### Choose an effect

| Effect | Behavior |
|--------|----------|
| **Audit** | Checks the requirement without blocking the save or automatically adding the baseline. |
| **Deny** | Enforces the requirement using the selected definition's Automatic or Manual behavior. This is the default effect. |
| **Disabled** | Turns off evaluation of this requirement. |

The Azure Policy **Deny** effect controls whether a guardrail configuration can be saved. A guardrail's **Block** action controls what happens to content at runtime. They aren't interchangeable settings.

## Step 1: Create the baseline guardrail

1. In a test Foundry resource, [create a guardrail](how-to-create-guardrails.md) with the built-in controls and custom rules that your organization requires.
1. Save it with a name that identifies its purpose, such as *organization-baseline*.
1. Assign it to a test model or agent and verify both content that should be blocked and content that should be allowed.
1. Record its full Azure Resource Manager resource ID:

   ```text
   /subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.CognitiveServices/accounts/<foundry-resource>/raiPolicies/<baseline-name>
   ```

Use the guardrail's resource ID, not its display name, an agent ID, or an ID inside its generated policy. Keep the baseline and the guardrails you test in the same Foundry resource for the initial setup. Manual comparison has additional resource-ownership checks for referenced moderation resources; check with your Microsoft contact before using a baseline from another Foundry resource.

## Step 2: Assign the Azure Policy definition

1. Open **Policy** in the [Azure portal](https://portal.azure.com).
1. Select **Definitions** and search for the full name of the Automatic or Manual definition from the table above.
1. Open the definition and select **Assign**.
1. Choose a test scope that contains only the intended Foundry resource, such as a resource group with no other Foundry resources. Review any exclusions so the target resource remains in scope.
1. On **Parameters**, enter the baseline's full resource ID in **ACS baseline ARM resource ID** (`policyId`).
1. Set **Effect** to **Audit** for the initial check. If the parameter isn't visible, clear **Only show parameters that need input or review**.
1. Review and create the assignment.

The built-in definition IDs are:

| Definition | ID |
|------------|----|
| Automatic | `9edf9b13-68ea-4cf8-8ac4-4e29ff113bdf` |
| Manual | `11f7e854-02de-41b2-a10e-0bad91fcea42` |

For background on assignment scopes and permissions, see [Azure Policy assignment structure](https://learn.microsoft.com/azure/governance/policy/concepts/assignment-structure) and [Azure Policy permissions](https://learn.microsoft.com/azure/governance/policy/overview#azure-rbac-permissions-in-azure-policy).

## Step 3: Verify the requirement and enable enforcement

1. Allow time for the assignment to propagate.
1. In the same test Foundry resource, create or edit a second guardrail with settings that don't meet the baseline. With **Audit**, the requirement shouldn't add the baseline or block the save.
1. Review the assignment and any available compliance results in **Policy**. Compliance reporting can lag behind configuration changes; a successful save under Audit doesn't prove that the requirement was evaluated.
1. Change **Effect** to **Deny** and ensure **Policy enforcement** is enabled on the assignment.
1. Allow the change to propagate, then repeat the create or update operation.
   - With **Automatic**, Foundry should save the guardrail with the baseline enforced alongside it. If Foundry can't enforce the baseline, the save can fail.
   - With **Manual**, a guardrail that doesn't preserve the baseline's blocking behavior should fail to save. Update its controls to satisfy the baseline, then save again. The accepted guardrail should save without adding the baseline as a separate runtime policy.
1. Assign the accepted guardrail to the test model or agent. Test content that the baseline blocks and content that it allows. Also test the team's additional custom rules.

Don't widen the assignment scope until you have verified both the save behavior and runtime enforcement. These definitions evaluate guardrail resources. An assignment alone doesn't create a guardrail, assign one to every model or agent, or prove that existing resources have been updated.

## Manage the baseline

Treat baseline edits as shared configuration changes. Test them before applying them to a wider scope, and recheck affected guardrails after an update.

Automatic enforcement retains a separate baseline requirement. Manual enforcement doesn't add that separate runtime layer: editing the baseline doesn't update a previously accepted guardrail's authored rules. Re-save and revalidate affected guardrails against the changed baseline.

Don't delete a baseline while assignments or saved guardrails still reference it. Coordinate a replacement with the teams that use it, update or remove the assignment, then re-save affected guardrails and verify their runtime behavior before deleting the old baseline. Previously saved Automatic guardrails can retain the old reference until you re-save them. Deleting the baseline first can cause policy resolution to fail.

## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| The definition is missing. | Search by the definition ID above. If you still can't find it, ask your Microsoft contact to confirm availability in your Azure cloud. |
| A save ignores the expected requirement. | Confirm preview support for the resource and region, assignment scope and exclusions, the effect, and whether assignment enforcement is enabled. Verify that the guardrail uses the new experience. |
| Audit lets a weaker guardrail save. | This is expected. Audit doesn't block saves or add the baseline. Use Deny after the test checks pass. |
| Manual enforcement rejects a guardrail that looks equivalent. | Compare the controls, intervention points, thresholds, judge deployment, and exact evaluation prompt. Check the tool configuration and references to blocklists or other moderation resources. Keep the error details for your Microsoft contact if the comparison can't be completed. Matching names alone doesn't prove equivalent behavior. |
| Content is blocked even after a team weakens its own rule. | Check for an automatically enforced baseline. It runs alongside the team's guardrail and can still block the content. |
| Behavior doesn't change as soon as you edit a policy or assignment. | Allow time for assignment and runtime configuration propagation, then repeat the save and runtime tests. Don't rely on immediate changes. |
| Saves fail across the assignment scope. | Check that `policyId` is the full resource ID of an existing baseline, starting with `/subscriptions/`. A display name or an ID copied from inside the generated policy isn't a valid baseline reference. |
| Saves or runtime requests fail after a baseline was moved or deleted. | Restore the referenced baseline or update the assignment to a valid replacement. Re-save and test affected guardrails so they no longer depend on the old reference. |

## Related content

- [Guardrails overview](overview.md)
- [Create and manage guardrails](how-to-create-guardrails.md)
- [Custom rules and LLM judges](custom-rules.md)
- [Limitations and troubleshooting](limitations.md)
