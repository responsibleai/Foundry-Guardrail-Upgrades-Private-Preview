# Quickstart: Block a tool call with a custom rule (private preview)

In this quickstart, you create a guardrail with a custom rule that stops an agent from calling a specific tool, assign the guardrail to the agent, and test it in the playground.

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features and behavior might change before general availability.

## Prerequisites

- Access to the guardrails private preview. To request access, see [Join the private preview](overview.md#join-the-private-preview).
- A [Microsoft Foundry project](https://learn.microsoft.com/azure/foundry/how-to/create-projects) with at least one model deployment.
- The **Foundry Account Owner** role or higher on the Foundry resource.
- A non-production agent in the project that uses at least one [supported tool](intervention-points.md#supported-tools). Note the name of a tool that you want to block.

## Create a guardrail with a custom rule

1. Go to [Foundry](https://ai.azure.com) and open your project.
1. Select **Build** in the upper-right menu, and then select **Guardrails** in the left navigation.
1. Select **Create guardrail**. The **Configure controls** step opens.
1. In the **Custom** section, under **Custom rules**, select **Add rule**.
1. In **Rule name**, enter *Block tool call*.
1. Next to **If**, select **Tool name** and **Equals**, and then enter the name of the tool that you want to block.
1. In **Intervention points**, select **Tool input**.
1. In **Action**, keep **Block**.
1. Select **Add rule**.
1. Keep the default built-in controls, and then select **Next**.

## Assign the guardrail to your agent

1. In the **Select agents and models** step, select **Add agents**.
1. Select your agent, and then select **Save**.
1. Select **Next**.
1. In the **Review** step, enter a name for the guardrail, such as *quickstart-tool-block*, and then select **Create**.

Allow time for the assignment to propagate, and then test the agent.

## Test the rule

1. Select **Agents** in the left navigation, and then select your agent to open it in the playground.
1. Send a request that causes the agent to call the tool that you blocked.

   The tool doesn't run. A message in the chat shows that the content was blocked, which control blocked it, and at which intervention point.

1. Send a request that doesn't need the blocked tool.

   The agent responds normally.

## Clean up resources

To restore the agent's previous behavior, assign a different guardrail to the agent, and then delete the quickstart guardrail. For steps, see [Delete a guardrail](how-to-create-guardrails.md#delete-a-guardrail).

## Next step

> [!div class="nextstepaction"]
> [Create and manage guardrails](how-to-create-guardrails.md)
