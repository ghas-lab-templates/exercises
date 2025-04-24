# 🚀 Supply chain security Track: Mission Critical Dependencies

## Scenario: The Case of the vulnerable vendor

Acme Corp, a fast-growing tech company, has recently faced a critical data breach due to a vulnerability in a third-party dependency. The incident has shaken customer trust and put the security team under immense pressure. As the newly appointed Supply Chain Security Champion, your mission is to secure Acme Corp's software supply chain and prevent future incidents.

## The stakes

Your success will determine whether Acme Corp can restore customer trust and avoid further reputational damage. The board is closely monitoring your progress, and your team is counting on you. No pressure! 😉

## Environment overview

You’ll receive a dedicated GitHub organization provisioned just for you. As the organization administrator, you have full access to:

- GitHub Advanced Security and all related Supply Chain Security products and features.
- GitHub Actions as your CI.
- GitHub.com with all its potential, meaning you have the freedom to implement your solutions using custom properties, security configurations, rulesets, secrets, etc., with Copilot right by your side throughout the journey.

You will drive this with the following principles in mind:

- **Risk-based approach**: Adopt rigorous controls for Business Critical Apps while keeping Standard and Low Risk repositories agile. This minimizes friction for developers and focuses effort where it matters most.
- **Centralized governance**: Central governance of configurations ensures a smooth, consistent rollout across all repositories. You can update policies from a single pane, reduce maintenance overhead, and quickly iterate security improvements. By automating settings and driving improvements centrally, you enable developers to move fast—securely—without manual overhead or roadblocks.

The organization contains a set of repositories you’ll configure and secure.

In `Module 0`, you’ll establish your baseline by:

- Reviewing and creating security configurations.  
- Defining a `Risk-level` custom property and tagging repositories (`Business Critical App`, `Standard`, `Low Risk`).  
- Creating an Advanced Security GitHub App for the organization to be used for third-party integrations, automation, and overall security management.

In `Module 1`, you’ll focus on Supply Chain Security by:

- Enabling and applying your custom security configuration to the current and future repositories.
- Exploring organization and repository Dependency Graphs.  
- Automating dependency submission and reviewing your SBOM.
- Implementing Dependabot Alerts, Rules, and Security Updates.
- Identifying the projects with the most vulnerabilities and prioritizing them for remediation.

## Objectives

Module references to guide you:

- Foundations: [Module 0](module-0.md)
  - Set up security configurations  
  - Tag and categorize repositories by risk  
  - Create a GitHub App and store its private key  

- Supply Chain Security: [Module 1](module-1.md)  
  - Enable Dependency Graph across all repositories  
  - Navigate organization and repository graphs to understand dependencies/licenses  
  - Automate SBOM generation and dependency submissions  
  - Implement Dependabot Alerts, Rules, Security, and Version Updates  
  - Implement Dependency Review and centralize it for all PRs for the highest risk repositories
  - Use the Security Overview dashboard to identify and prioritize projects with the most vulnerabilities.

## References

- [GitHub supply chain security](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-supply-chain-security)
- [GitHub security features](https://docs.github.com/en/code-security/getting-started/github-security-features)
- [Security Overview](https://docs.github.com/en/code-security/security-overview/about-security-overview)
- [GitHub Actions](https://docs.github.com/en/actions)
- [GitHub Apps](https://docs.github.com/en/apps/overview)
- [Organization rulesets](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization)
- [Security configurations](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/enabling-security-features-in-your-organization/creating-a-custom-security-configuration)
- [Custom properties](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization)