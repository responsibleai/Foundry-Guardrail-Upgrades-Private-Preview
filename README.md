# Microsoft Foundry guardrails with custom rules (private preview)

This repository contains product documentation for the private preview of the new guardrail experience in Microsoft Foundry. In this experience, you protect your models and agents by combining Microsoft's built-in safety and security controls with custom rules that you define for your own business policies.

The articles don't assume that you've used Foundry guardrails before.

> [!IMPORTANT]
> This feature is in private preview. It's provided without a service-level agreement and isn't recommended for production workloads. Features, user interface labels, and behavior might change before general availability.

## Who this documentation is for

This documentation is for enterprise developers, AI platform teams, and security and compliance teams who build models and agents in Microsoft Foundry and need to enforce safety, security, and organization-specific policies.

## Recommended reading order

| Order | Article | What you learn |
|-------|---------|----------------|
| 1 | [Guardrails overview](docs/overview.md) | What guardrails are, how they evaluate requests, and everything you can do in the new experience. |
| 2 | [Intervention points](docs/intervention-points.md) | The four stages of an interaction where guardrails check content. |
| 3 | [Built-in controls](docs/built-in-controls.md) | The safety and security risks that Microsoft classifiers detect, and how to configure them. |
| 4 | [Custom rules and LLM judges](docs/custom-rules.md) | How to express your own policies with conditions and LLM judges. |
| 5 | [Quickstart: Block a tool call with a custom rule](docs/quickstart-custom-rule.md) | A short, end-to-end walkthrough. |
| 6 | [Create and manage guardrails](docs/how-to-create-guardrails.md) | Complete portal procedures for creating, assigning, testing, editing, and deleting guardrails. |
| 7 | [Govern guardrails with Azure Policy](docs/azure-policy.md) | Require an organization baseline, choose Automatic or Manual enforcement, and verify the result. |
| 8 | [Limitations and troubleshooting](docs/limitations.md) | Preview limitations and solutions to common issues. |

If you want hands-on experience first, start with the [quickstart](docs/quickstart-custom-rule.md), and then return to the concept articles.

## Join the private preview

To join, complete the [private preview invitation form](https://aka.ms/agentcontrolssignups). The Microsoft engineering team contacts you and sets up the preview for you.

## Provide feedback

Share feedback and report issues through the private preview channel that your Microsoft contact provided.

## Contributing

This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit [Contributor License Agreements](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot automatically determines whether you need to provide a CLA and decorates the PR appropriately (for example, with a status check or comment). Follow the instructions provided by the bot. You only need to do this once across all repos using our CLA.

For details, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Code of conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Security

**Don't report security vulnerabilities through public GitHub issues.** For security reporting information, locations, contact information, and policies, see [SECURITY.md](SECURITY.md) and the latest guidance for Microsoft repositories at [https://aka.ms/SECURITY.md](https://aka.ms/SECURITY.md).

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow [Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/legal/intellectualproperty/trademarks/usage/general). Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship. Any use of third-party trademarks or logos is subject to those third parties' policies.

## Legal notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode). See the [LICENSE](LICENSE) file.

Privacy information can be found at [https://privacy.microsoft.com](https://privacy.microsoft.com).

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents, or trademarks, whether by implication, estoppel, or otherwise.
