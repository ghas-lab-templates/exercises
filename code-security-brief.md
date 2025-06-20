# 🛡️ Code Security Track: Securing the Codebase

## Scenario: The Case of the Vulnerable Code

Acme Corp, a fast-growing tech company, has recently uncovered critical vulnerabilities in their codebase through a third-party security audit. These findings have raised concerns about the overall security posture of their applications and put the development team under scrutiny. As the newly appointed Code Security Champion, your mission is to implement comprehensive code security practices and prevent future vulnerabilities from reaching production.

## The stakes

Your success will determine whether Acme Corp can maintain its reputation for secure, high-quality software. The executive team is closely monitoring your progress, and developers across the organization are looking to you for guidance. No pressure! 😉

## Environment overview

You'll receive a dedicated GitHub organization provisioned just for you. As the organization administrator, you have full access to:

- GitHub Advanced Security and all related Code Security products and features.
- GitHub Actions as your CI.
- GitHub.com with all its potential, meaning you have the freedom to implement your solutions using custom properties, security configurations, rulesets, secrets, etc., with Copilot right by your side throughout the journey.

You will drive this with the following principles in mind:

- **Risk-based approach**: Adopt rigorous controls for Business Critical Apps while keeping Standard and Low Risk repositories agile. This minimizes friction for developers and focuses effort where it matters most.
- **Centralized governance**: Central governance of configurations ensures a smooth, consistent rollout across all repositories. You can update policies from a single pane, reduce maintenance overhead, and quickly iterate security improvements. By automating settings and driving improvements centrally, you enable developers to move fast—securely—without manual overhead or roadblocks.

The organization contains a set of repositories you'll configure and secure.

In `Module 0`, you'll establish your baseline by:

- Reviewing and creating security configurations.  
- Defining a `Risk-level` custom property and tagging repositories (`Business Critical App`, `Standard`, `Low Risk`).  
- Creating an Advanced Security GitHub App for the organization to be used for third-party integrations, automation, and overall security management.

In `Module 3`, you'll focus on Code Security by:

- Enabling and configuring code scanning across all repositories based on risk level.
- Setting up default code scanning configurations for different repository types.
- Exploring code scanning results and reviewing security vulnerabilities.
- Implementing automated fix suggestions for common vulnerabilities.
- Setting up mandatory code scanning checks for high-risk repositories.
- Using the Security Overview dashboard to track and prioritize code security issues.

## Objectives

Module references to guide you:

- Foundations: [Module 0](module-0.md)
  - Set up security configurations  
  - Tag and categorize repositories by risk  
  - Create a GitHub App and store its private key  

- Code Security: [Module 3](module-3.md)  
  - Enable code scanning across all repositories with risk-appropriate configurations
  - Configure default code scanning settings for different repository types
  - Review and triage code scanning alerts
  - Implement automated code scanning fix suggestions
  - Set up mandatory code scanning checks for pull requests in high-risk repositories
  - Use the Security Overview dashboard to identify and prioritize code security issues
  - Customize code scanning with additional queries and rules to catch organization-specific vulnerabilities

## References

- [GitHub code security](https://docs.github.com/en/code-security/getting-started/github-security-features)
- [Code scanning](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning)
- [Security Overview](https://docs.github.com/en/code-security/security-overview/about-security-overview)
- [GitHub Actions](https://docs.github.com/en/actions)
- [GitHub Apps](https://docs.github.com/en/apps/overview)
- [Organization rulesets](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization)
- [Security configurations](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/enabling-security-features-in-your-organization/creating-a-custom-security-configuration)
- [Custom properties](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization)
