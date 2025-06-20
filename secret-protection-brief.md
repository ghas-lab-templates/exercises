# 🔒 Secret Protection Track: Securing Sensitive Data

## Scenario: The Case of the Leaked Credentials

Acme Corp, a fast-growing tech company, has recently discovered that API keys and database credentials were accidentally committed to their public repositories and leaked. This incident has compromised critical infrastructure and put the security team in damage control mode. As the newly appointed Secret Protection Champion, your mission is to implement comprehensive secret detection and prevention measures to ensure sensitive data never leaks again.

## The stakes

Your success will determine whether Acme Corp can regain control of their sensitive information and prevent future credential exposures. The CISO is directly overseeing your progress, and the entire organization is counting on you to establish secure practices. No pressure! 😉

## Environment overview

You'll receive a dedicated GitHub organization provisioned just for you. As the organization administrator, you have full access to:

- GitHub Advanced Security and all related Secret Protection products and features.
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

In `Module 2`, you'll focus on Secret Protection by:

- Enabling and configuring secret scanning across all repositories.
- Setting up push protection to prevent secrets from being committed.
- Creating custom secret scanning patterns for organization-specific credentials.
- Implementing automated response workflows for detected secrets.
- Setting up notifications and alerts for security teams.
- Using the Security Overview dashboard to monitor secret scanning alerts across repositories.

## Objectives

Module references to guide you:

- Foundations: [Module 0](module-0.md)
  - Set up security configurations  
  - Tag and categorize repositories by risk  
  - Create a GitHub App and store its private key  

- Secret Protection: [Module 2](module-2.md)  
  - Enable secret scanning across all repositories with risk-appropriate configurations
  - Configure push protection to block secret commits in real-time
  - Create custom secret scanning patterns for organization-specific credentials
  - Implement automated secret rotation workflows when secrets are detected
  - Set up alerting for the security team when critical secrets are found
  - Use the Security Overview dashboard to track secret scanning alerts
  - Define custom remediation guidance for repository maintainers

## References

- [GitHub secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
- [Push protection](https://docs.github.com/en/code-security/secret-scanning/protecting-pushes-with-secret-scanning)
- [Custom patterns for secret scanning](https://docs.github.com/en/code-security/secret-scanning/defining-custom-patterns-for-secret-scanning)
- [Security Overview](https://docs.github.com/en/code-security/security-overview/about-security-overview)
- [GitHub Actions](https://docs.github.com/en/actions)
- [GitHub Apps](https://docs.github.com/en/apps/overview)
- [Organization rulesets](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization)
- [Security configurations](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/enabling-security-features-in-your-organization/creating-a-custom-security-configuration)
- [Custom properties](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization)
