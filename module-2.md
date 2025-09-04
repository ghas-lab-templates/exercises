
# Module 2 - Secret Scanning

## Enablement  

### Lab 1 - Setting Up a Custom Security Configuration and Enabling Secret Scanning

#### Objective 
In this lab, you will learn how to create and apply a custom security configuration to repositories in your organization. 

#### Steps

1. In GitHub, navigate to the security configuration you created in module 1.
2. For each configuration option under `Secret Scanning` select the following:
- `Secret Scanning - Alerts`: Select `Enabled`
- `Validity check`: Select `Enabled`
- `Non-provider patterns`: Select `Enabled`
- `Scan for generic passwords`: Select `Enabled`
- `Push protection`: Select `Enabled`
- `Bypass priveleges`: Select `Not set`
- `Prevent direct alert dismissals`: Select `Enabled`
3. Click on `Save Configuration` button. Please confirm save if prompted.
4. Navigate to the `mona-gallery`. Under the `Security` tab, select `secret scanning`. Inspect the alerts. 

### Lab 2 - Custom Patterns with AI 

#### Objective

The goal of this lab is to demonstrate how to use custom patterns in GitHub to identify secrets that are not part of the Secret Scanning Partner Program.

#### Steps
1. Navigate to the `mona-gallery` repository in your GitHub Organization
2. Please note that Custom patterns can be created at 3 levels within the GitHub hierarchy - Enterprise, Organization & Repository. For this Lab, we will create the custom pattern in the repo scope
3.Open the repository’s Settings tab. Under the `Code security` section, go to `Custom Patterns` and click `New pattern`.
4. In the top right hand corner, click on `Generate with AI`

  <details>
    <summary> Animated Guide</summary>

  ![alt text](images/custom-pattern-with-ai.gif)

  </details>

5. A custom secret has been embedded in the file: `storage/src/main/resources/application.properties`
6. We will use AI assistance to generate a regular expression for the secret `minio.password` mapped to the value `mona_value_abc124`
7. Fill in the options `I want a regular expression that` and `Examples of what I am looking for`
8. Once the generated regular expression meets your needs, click `Use result`
9. This copies the generated regular expression and test strings into the New Custom Pattern creation page.

  <details>
    <summary> Animated Guide</summary>

  ![alt text](images/custom-pattern-prompt.gif)

  </details>

10. Provide a clear and descriptive Pattern name, then click Save and dry run.

  <details>
    <summary> Animated Guide</summary>

  ![alt text](images/custom-pattern-publish.gif)

  </details>

### Lab 3 - Push Protection 

#### Objective

The purpose of this lab is to demonstrate and familiarize participants with the Secret Scanning Push Protection feature in GitHub.

#### Steps
1. Navigate to the `mona-gallery` repository in your GitHub Organization
2. Recall from Lab 2 that we created and published a custom pattern to capture the secret `mona_value_abc124`.
3. Commit changes to the source code that include a string matching the custom pattern.
4. Oncel, push protection is enabled for the custom pattern, we will then try to commit some changes in the source code by trying to commit a string which matches the custom pattern
5. Because Push Protection is active for the repository, the commit will be blocked, preventing the secret from being pushed.

  <details>
    <summary> Animated Guide</summary>

  ![alt text](images/custom-pattern-publish.gif)

  </details>

### Lab 4 - Custom Patterns at Organization level 

#### Objective 

In this lab, you will learn how to create a GitHub Secret Scanning custom pattern at the organization level to detect leaked Bearer tokens. You will identify common patterns, and write a regex that accurately matches real secrets. You will also explore the features and limitations of regular expressions, and practice striking the right balance between detecting true positives and minimizing false positives. Throughout the lab, you will use the different matching criteria available in GitHub Secret Scanning to refine and improve your detection pattern.

Bearer tokens are a type of access token commonly used in modern APIs and authentication systems. They allow users, applications, or services to prove their identity and gain access to protected resources without needing to repeatedly provide a username and password.

Bearer tokens are typically short-lived and are passed in the Authorization header of HTTP requests, using the format `Authorization: Bearer <token>`. If a bearer token is leaked, anyone who obtains it can use it to impersonate the original user or system, often with full access to sensitive data or functionality.

In this lab, you will focus specifically on identifying leaked Bearer tokens by analyzing their structure and writing a custom regex pattern to detect them while minimizing false positives.

#### Steps

1. Go to your GitHub organization’s `Settings` > `Advanced Security` > `Global Settings`. Scroll to the `Custom patterns` section under `Secret Scanning` . Click on `New pattern`. 
2. Name the pattern, this can be anything, but it should clearly indicate what the pattern is scanning for — for example, `Bearer Token`.
3. First, define the pattern for the token itself — the secret that needs to be detected —  in the `Secret format` section. Since Bearer tokens can vary widely, but usually appear as long, random-looking strings, you should match a sequence of characters commonly found in secrets. This includes uppercase and lowercase letters, numbers, and special characters such as _, -, +, /, and =. To avoid matching regular English words or short random fragments, you should require the token to be at least 15 characters long. A simple character class with a minimum length ensures the token looks like a real secret and not regular text.

  <details>
    <summary>Solution</summary>

    ```md
      [a-zA-Z0-9_\-+=/.]{15,}
    ```
  </details>

4. Next, match what typically appears before a Bearer token in the `Before secret` section. In most cases, a Bearer token is introduced by an HTTP header like Authorization: Bearer <token>, or it might appear inside a quoted string in source code, such as "Bearer <token>". Your regex should account for these common formats. To do this, you should match either the word Authorization: followed by optional spaces, or a single or double quote character. After that, you should match the word Bearer (or Token) in a case-insensitive way, followed by at least one space. This ensures you are correctly targeting contexts where a Bearer token is presented.

<details>
  <summary>Solution</summary>

   ```md
    (Authorization:\s*|['"])((?i)Bearer\s+|(?i)Token\s*(token=)?)
   ```
</details>

5. After matching the token, you must ensure that the match ends cleanly in the `After Secret` secion. Typically, a real token will end with a space, a quotation mark, or at the very end of a line or file. Your regex should enforce that after the token, there is either whitespace, a quote, or the end of input. This helps prevent accidentally matching extra text or trailing characters that do not belong to the token.

  <details>
    <summary>Solution</summary>

    ```md
      \z|[\s'"]
    ```
  </details>

6. Before you can `Save and dry run` you must provide a `Test string`. You can use this one to get a match `Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`. Once provided you can `Save and dry run`. You will be prompted to select the repositories you want to run the dry run on. Select `All repositories in the organization`.
7. Once satisfied with dry run results, click on `Publish pattern`. 

#### Discussion Points
- What factors might cause false positives when detecting Bearer tokens using regex?
- How can you balance accuracy (true positives) against sensitivity (false positives) in your regex?
- What limitations or edge cases might your regex pattern fail to detect?
- How would you approach updating or maintaining custom patterns as new token formats emerge?
- What challenges might arise when applying custom secret scanning patterns across diverse teams or projects within your organization?

### Lab 5 - Secret Remediation  

#### Objective 

In the previous lab, you set up a custom GitHub Secret Scanning pattern that successfully detected a leaked Bearer token in the repository `github-configurations`, located in the file `.github/workflows/secret-scanning-report.yml` on line 25.

In this lab, you will go through the correct steps to remediate this leaked secret to prevent unauthorized access or misuse.

#### Steps
1. Navigate to the Security tab of the `github-configurations` repository and select Secret scanning alerts. Locate the alert that was triggered for `.github/workflows/secret-scanning-report.yml` on line 25. Open the file in your repository, find the leaked token on the specified line, and carefully review the alert details. It is important to understand that even if the token has not been actively misused yet, it must be treated as compromised.
2. Identify the source and usage of the token. Determine whether the token is tied to a GitHub PAT, an OAuth integration, a CI/CD system, or another external service. Once you have identified the system associated with the token, revoke it immediately. This is a dummy token so not action needed. In a real-world scenario, if you manage the token, revoke it from the issuing platform yourself; otherwise, escalate it to the relevant service owner or security team. For GitHub tokens, you can navigate to Settings > Developer settings > Personal access tokens to find and revoke the compromised token.
3. After revoking the old token, you must generate a new one. Create a replacement GitHub PAT token with only the minimum permissions necessary for its intended use. Click on your profile in the top right hand corner, and click on `Settings` in the menu bar. Scroll down to the bottom and select `Developer settings`. Under `Personal access tokens` select `Tokens(classic)` and generate a new classic token.
4. Store this new token securely, such as using GitHub Actions Secrets. Navigate to a the repository with the workflow, `github-configurations`, and under `Settings` select `Actions` under the `Secrets and variables` section. Click on `New repository secret` and add in your newly created PAT and call it `SECRET_SCANNING_TOKEN`. Note that you can use any secret management system to store and use your workflow secrets like HashiCorp Vault, or an encrypted key vault provided by your cloud provider. The important lesson here is to **never commit the new token directly into your repository**.
5. Update the GitHub Actions to reference the new secret  
  <details>
    <summary>Solution</summary>

    ```yaml
        env:
          GITHUB_TOKEN: ${{ secrets.SECRET_SCANNING_TOKEN }}
    ```
  </details>

6. Verify the actions works and close the secret scanning alert with reason `Revoked`. 

#### Discussion Points
- In what scenarios would you also re-write git history as a part of the secret remediation process?
- What strategies or practices can help prevent accidental leakage of tokens in repositories?
- Why is it important to assign minimal permissions to tokens, even if securely managed?
- How would you coordinate secret remediation efforts across multiple teams or services?
- What role does organizational culture play in preventing and responding effectively to secret leaks?

### Lab 6 - Webhooks   

#### Objective 
In this lab, you will configure a webhook for the mona-gallery repository to listen specifically for Secret Scanning alerts. You will use smee.io as a temporary webhook delivery service to receive and inspect the payloads.
This will allow you to understand how GitHub sends secret scanning events and prepare you for building automated responses to leaked secrets in a real-world environment.

#### Steps
1. Navigate to https://smee.io/ and click Start a new channel. Copy the generated URL — you will use this as the payload URL for the webhook.
2. Create a webhook in the mona-gallery repository.
Navigate to the mona-gallery repository. Go to `Settings` > `Webhooks` > `Add webhook`.
In the webhook setup form:
  - Payload URL: Paste the Smee.io channel URL you generated.
  - Content type: Select application/json.
  - Secret: Leave blank for this test (in production, always set a secret).
  - Under Which events would you like to trigger this webhook?, choose Let me select individual events.
  - Select only the checkbox for Secret scanning alert.
  - Save the webhook.
3. Navigate to the secret scanning alerts, and choose the alert for the Bearer token. Close this alert as `False Positive`. 
4. Inspect incoming webhook payloads on Smee.io.
5. Open your Smee.io channel page. Youu should see incoming JSON payloads whenever GitHub sends a Secret Scanning event. Review the fields such as `alert_type`, `location`, `repository`, and `state`.

#### Discussion Points
- What considerations should you keep in mind when configuring webhooks at scale across many repositories?
- How would you integrate webhook-based alerting with existing security tools or incident management platforms?
- What kinds of automated actions could your organization take when a secret leak is detected?

