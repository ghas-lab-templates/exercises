# Module 1 - Supply Chain Security

## Enablement  

### Lab 1 - Editing your Security Configuration and Enabling Dependency Graph

#### Objective 
In this lab, you will learn how to apply a custom security configuration to repositories in your organization.

#### Steps

1. Click on your organization's settings. In the `Security` section of the sidebar, select the `Advanced Security` dropdown menu, then click `Configurations`. You will be navigated to the `Code security configurations` page. By now you are familiar with this page, so go ahead and click the edit (pencil) icon next to the two configuration you created earlier in Module 0.
2. For each configuration option select the following:
 - `GitHub Advanced Security Features`: Select `Include`.
 - Under the `Dependency Scanning` section:
  - `Dependency Graph`: Select `Enabled`.
  -  All Other Settings: Select `Not set`.
 - In the `Policy` options, for `Use as default for newly created repositories`, select `All repositories`.
 - In the `Policy` options, for `Enforce Configuration`, select `Don't Enforce`.

4. Click on the `Save Configuration` button. Please confirm save if prompted.

  <details>
 <summary>Need Help? View Configuration Screenshot</summary>  
   
   ![alt text](images/configuration-options.png)
   
 </details>

5. The page will be redirected to the `Configurations` page.
  - For the `Supply chain security - Basic` configuration:
    - Click on the `Apply to` dropdown and select `All repositories`. There will be a prompt for confirmation; select `Apply`.
  - For the `Supply chain security - Business Critical` configuration:
    - You can use the other way to apply configuration to subset of repositories. Select the `custom property` and the then select the `Risk-level` custom propertyh with `Business Critical App` value. This should show the `mona-gallery` repository as a search result. Select the `mona-gallery` repository and using the `Apply configuration` button, apply the `Supply chain security - Business Critical` configuration to it.

<details>
  <summary>Animated Guide</summary>
  
**TO DO: GIF THIS WHEN TOTAL NUMBER OF REPOS CONFIRMED**

![alt text](images/applytoallrepos.png)
</details>

### Discussion Points
- What is the difference between enforced and non-enforced configurations? What is the order of precedence?

## Supply Chain Security - Know Your Environment

### Lab 2 - Dependency Graph

#### Objective 

Understand how to navigate and interpret dependency graphs at both the organization and repository levels. Gain insights into the licenses used and the relationships between dependencies and dependents to ensure comprehensive dependency management.

#### Steps: Organization Level Dependencies 
1. On the Organization page, locate the `Insights` tab in the navigation bar at the top. Under the `Insights` sections, find and click on `Dependencies` from the left-hand menu. 
2. Review the licenses used in the Organization.
3. Explore the relationship between dependencies and dependents.
<details>
  <summary> Animated Guide</summary>

![alt text](images/org-dependency-graph.gif)

</details>

#### Steps: Repository Level Dependencies 
1. Navigate to the `mona-gallery` repository in your GitHub Organization.
1. On the repository page, locate the `Insights` tab in the navigation bar at the top. Under the `Insights` sections, find and click on `Dependency Graph` from the left-hand menu. 
2. Carefully review the list of dependencies displayed and verify completeness. Look for any missing dependencies.
<details>
  <summary> Animated Guide</summary>

![alt text](images/repo-dependency-graph.gif)

</details>

#### Discussion Points 
- How do dependency relationships (dependencies vs. dependents) impact project maintenance and scalability?
- What tools or practices can be used to ensure dependency graphs remain accurate and up-to-date?

### Lab 3 - Dependency Submission API

#### Objective
In this lab, we'll learn how to use the Dependency Submission Action to correctly populate the dependency graph.

#### Steps
1. Navigate to the `moshi` repository in your GitHub Organization.
2. Notice that the dependency graph only shows 3 dependencies. This is unusual for a project of its size because the repository uses Gradle for its build process and Gradle resolves dependencies dynamically during build time.
3. Add the Dependency Submission Action. Fortunately, Gradle provides a GitHub Action that can generate and submit a dependency graph for Gradle projects.
  a. Navigate to the `.github/workflows` directory in your repository and create the `dependency-submission.yml` file.
  b. Copy and paste the following workflow into the file:

``` yaml

name: Dependency Submission

on:
  push:
    branches: [ 'master' ]
  pull_request:
    branches: ['master']

permissions:
  contents: write

jobs:
  dependency-submission:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout sources
      uses: actions/checkout@v4
    - name: Configure JDK
      uses: actions/setup-java@v4
      with:
          distribution: 'zulu'
          java-version: '21'
          cache: 'gradle'

    - name: Generate and submit dependency graph
      uses: gradle/actions/dependency-submission@v4
```

4. Commit and push the changes to the repository.

  <details>
    <summary>Git Commands </summary>

  ```bash
  git add .github/workflows/dependency-submission.yml
  git commit -m "Add dependency submission workflow for Gradle"
  git push
  ```

  </details>

5. Ensure the workflow runs successfully. You can verify this in the `Actions` tab of the repository.
6. Once the workflow completes, navigate back to `Insights` > `Dependency graph`.
7. Confirm that the dependency graph now shows a complete and accurate list of dependencies.

#### Discussion Points
-  Why is it important to have a complete and accurate dependency graph? How can incomplete graphs affect project security and maintenance?
- How does automating dependency submission improve workflow efficiency compared to manually tracking dependencies? Are there any drawbacks?
- How should your team incorporate reviewing and maintaining dependency graphs into their regular workflows?

### Lab 4 - Automatic Dependency Submission

#### Objective

In this lab, we'll learn how to use the built-in Automatic Dependency Submission feature to correctly populate the dependency graph. Instead of adding an Actions workflow as in the previous exercise, we will use the built-in feature to automatically submit dependency graphs for supported languages.

#### Steps
1. Navigate to the `mona-gallery` repository in your GitHub Organization.
2. Filter the dependencies by `ecosystem:Macen`.
3. Notice the dependency graph only shows 4 dependencies. This is unusual for a project of its size. 
4. Instead of adding an Actions workflow like in the previous exercises, we will use the built-in feature to automatically submit dependency graphs for supported languages. We can do this by enabling the `Automatic Dependency Submission` feature in the repository settings or better yet continue using the custom security configuration we created earlier.
5. Click on your organization's settings. In the `Security` section of the sidebar, select the `Advanced Security` dropdown menu, then click `Configurations`. Locate the custom configuration you created earlier to edit it.
6. Under the `Dependency scanning` section, navigate to the dropdown for `Automatic dependency submission` and set it to `Enabled`.
8. Navigate back to the `mona-gallery` repository and confirm that the changes made have been applied by looking at the `Settings` > `Code Security`. The `Automatic dependency submission` should be set to `Enabled`.
9. The automatic submission will occur on the first push to the pom.xml file after the option is enabled. Go ahead and make a small change to the `storage/pom.xml` file – you can add a comment or change the version of one of the dependencies.
10. Give it a minute for the workflow to finish before confirming that the dependency graph now shows a complete list of Maven dependencies, both direct and transitive.

#### Discussion Points
-  What are the advantages of using the built-in Automatic Dependency Submission feature over manually submitting dependency graphs?
-  Why is it important to have a complete and accurate dependency graph? How can incomplete graphs affect project security and maintenance?
- How does the built-in Automatic Dependency Submission feature compare to using the Dependency Submission Action in terms of ease of use and integration with existing workflows?

### Lab 5 - Software Bill Of Materials (SBOM) Generation and Attestations

#### Objective 

A Software Bill of Materials (SBOM) is a comprehensive list of software components, dependencies, and versions within a project. Generating an SBOM helps improve supply chain security and enables transparency about the software used.

Methods to Retrieve SBOM:
- UI: You can download an SBOM directly via GitHub's UI under the Dependency graph or Security features.
- REST API: GitHub provides a REST API to retrieve SBOMs programmatically.
- GitHub Action: For automation, the SBOM Generator Action is a wrapper around the REST API and integrates directly into your workflows.

In this lab, we will use a GitHub Action to generate, upload, and attest the SBOM.

#### Steps
1. Navigate to the `mona-gallery`repository in your organization.
2. Create a new workflow file named `generate-sbom.yml` in the `.github/workflows` directory. 
3. Add the following contents to the file: 

```yaml

name: SBOM Generation and Attestation

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: write
      id-token: write
      attestations: write
      packages: write

    steps:
      # Step 1: Checkout code
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Generate SBOM
      - name: Generate SBOM
        uses: advanced-security/sbom-generator-action@v0.0.1
        id: sbom
        env: 
          GITHUB_TOKEN: ${{ github.token }}

      # Step 3: Upload SBOM as an artifact
      - name: Upload SBOM Artifact
        uses: actions/upload-artifact@v4
        with:
          path: ${{ steps.sbom.outputs.fileName }}
          name: "SBOM"
```

5. SBOM attestation ensures the integrity, authenticity, and trustworthiness of a Software Bill of Materials by proving it was generated from a secure and verified process. Update the workflow to use the attest-build-provenance action to create an attestation for our SBOM.

```yaml 
  - uses: actions/attest-build-provenance@v2
      with:
       subject-path: ${{steps.sbom.outputs.fileName }}
```

<details>
  <summary> Solution: workflow file</summary>

```yaml
name: SBOM Generation and Attestation

on:
  push:
    branches: [ "main" ]

jobs:
 build:
   runs-on: ubuntu-latest
   permissions:
     actions: read
     contents: write
     id-token: write
     attestations: write
     packages: write
    
   steps:
    - uses: actions/checkout@v3

    - uses: advanced-security/sbom-generator-action@v0.0.1
      id: sbom
      env: 
        GITHUB_TOKEN: ${{ github.token }}
        
    - uses: actions/attest-build-provenance@v2
      with:
       subject-path: ${{steps.sbom.outputs.fileName }}

    -  uses: actions/upload-artifact@v3.1.0
       with: 
        path: ${{steps.sbom.outputs.fileName }}
        name: "SBOM"
```

</details>

#### Discussion Points 
- Why are SBOMs critical for supply chain security? How can they help organizations manage software vulnerabilities and risks?
- How does SBOM attestation ensure the integrity and authenticity of the generated SBOM? What are the implications of not attesting SBOMs?
- How can SBOMs assist in meeting compliance requirements and providing transparency to stakeholders or customers?
- What challenges might teams face in generating, managing, and using SBOMs effectively? How can these challenges be addressed?
- How can teams integrate SBOM generation and attestation into their CI/CD pipelines without causing disruptions?
- What strategies should be used to store and manage SBOM artifacts securely and efficiently? How can teams ensure they are accessible when needed?

## Supply Chain Security - Manage Your Environment

### Lab 6 - Dependabot Alerts and Security Updates

#### Objective

In this lab, we will enable and review Dependabot Alerts and Dependabot Security Updates.

#### Steps
1. Edit your custom security configuration to enable Dependabot. Click on your organization's settings. In the `Security` section of the sidebar, select the `Advanced Security` dropdown menu, then click `Configurations`. Locate the custom configuration you created earlier. Click on the Edit (pencil) icon next to the configuration.
2. Under the `Dependency scanning` section, navigate to the dropdowns for `Dependabot alerts` and `Security updates`, and set both to `Enabled`.
3. Navigate back to the `mona-gallery` repository and check in the `Settings` > `Code Security` section if the changes have been applied.
4. Navigate to the `Actions` tab and check if there are any `Dependabot Updates` workflows in progress. Review the workflow logs to see if the update was successful.
5. Navigate to the `Pull requests` tab and check if there are any open pull requests from Dependabot. Review the pull requests.
9. Merge a PR. After merging, check the `Security` tab again to see if the alert has automatically closed.

#### Discussion Points
- What factors should you consider when setting up this automation, such as potential disruptions, update frequency, and testing the changes?
- How do your developers feel about receiving automatic pull requests? Are they concerned about workload, stability, or reviewing frequent updates?
- How will you triage the alerts and prioritize them for review (e.g., critical vulnerabilities first)?
- Who in your team or organization will be responsible for managing these alerts and reviewing the pull requests?
- In what situation would you use group dependabot updates? What are the pros and cons? 

### Lab 7 - Dependabot Rules

#### Objective
In this lab, we will configure Dependabot Rules to manage alerts in the `mona-gallery` repository. Since this repository is very active and the development team does not have time to remove or replace dependencies unless a patch is available, we will create a rule to dismiss alerts for the `Go` and `Pip` ecosystems when no patch exists.

#### Steps
1. Navigate to the `mona-gallery` repository. Click on the `Settings` tab, under `Advanced Security`, select `Code security`.
2. Scroll to the `Dependabot Rules` section and select the cog wheel icon to manage rules. Click on the `New Rule` green button.
3. Set Up the Rule:
- Rule Name: Dismiss No-Patch Alerts for Go and Pip.
- Target Alerts > Ecosystem: Select Go and Pip from the dropdown list.
- Rules : Select Dismiss alerts > Until patch is available.
4. Select `Create Rule` button.
5. Navigate to the Dependabot Alerts tab and view which alerts have been closed. 

#### Discussion Points
- What are the potential risks of dismissing alerts for unpatched vulnerabilities? How can these risks be mitigated?
- How should teams prioritize alerts that cannot be dismissed due to the presence of patches? What criteria should be used to decide on the urgency of addressing these vulnerabilities?
- How can teams effectively communicate the rationale for dismissing certain alerts to stakeholders, including management and security teams?

### Lab 8 - Dependabot Version Updates

#### Objective

In this exercise, we will create a Dependabot configuration file to automate version updates for npm dependencies in the frontend directory of the `mona-gallery` repository.

#### Steps

1. Inside the `.github` folder create a `dependabot.yml` file.
2. Add the following configuration (Note: you must first create the label for this workflow to work).

```yaml

version: 2
updates:
  # Keep npm dependencies up to date
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
    # Raise all npm pull requests with custom labels
    labels:
      - "dependabot-version-update"
    commit-message:
      prefix: "DEPENDABOT VERSION UPDATES"
    
```

3. Commit and push your changes.


<details>
    <summary>Git Commands </summary>

  ```bash
git add .github/dependabot/dependabot.yml
git commit -m "Add Dependabot configuration for npm updates"
git push origin main
  ```
  </details>

4. Navigate to the `Pull requests` tab and review the generated pull requests.

#### Discussion Points 
- What factors influence the decision to schedule updates weekly, daily, or monthly?
- How can you balance the frequency of updates with the workload of reviewing pull requests?
- Discuss managing dependency updates across multiple teams and projects.
- How can you test the configuration to ensure it is working as expected?
- What is the process for reviewing and merging Dependabot pull requests?
- How can you automate testing for these pull requests to ensure they don’t break your code?
- How do you handle deprecated or unmaintained dependencies?


### Lab 9 - Dependency Review

#### Objective

In this lab, we will configure a Dependency Review workflow in GitHub Actions to analyze pull requests (PRs) and alert on the introduction of vulnerable dependencies to the `mona-gallery` repository.

#### Steps
1. Open your repository in Codespaces.
2. Create a new file named `dependency-review.yml` in the `.github/workflows` directory 
3. Add the following configuration file: 

```yaml

name: 'Dependency Review'
on: [pull_request]

permissions:
  contents: read
  pull-requests: write

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
    - name: 'Checkout Repository'
      uses: actions/checkout@v4
    - name: Dependency Review
      uses: actions/dependency-review-action@v4
      with:
        # Possible values: "critical", "high", "moderate", "low"
        fail-on-severity: moderate       
        # You can only include one of these two options: `allow-licenses` and `deny-licenses`
        # ([String]). Only allow these licenses (optional)
        # Possible values: Any SPDX-compliant license identifiers or expressions from https://spdx.org/licenses/
        allow-licenses: GPL-3.0, BSD-3-Clause, MIT
        # ([String]). Block the pull request on these licenses (optional)
        # Possible values: Any SPDX-compliant license identifiers or expressions from https://spdx.org/licenses/
        #deny-licenses: LGPL-2.0, BSD-2-Clause
        comment-summary-in-pr: true
        # ([String]). Skip these GitHub Advisory Database IDs during detection (optional)
        # Possible values: Any valid GitHub Advisory Database ID from https://github.com/advisories
        allow-ghsas: GHSA-abcd-1234-5679, GHSA-efgh-1234-5679
        # ([String]). Block pull requests that introduce vulnerabilities in the scopes that match this list (optional)
        # Possible values: "development", "runtime", "unknown"
        fail-on-scopes: development, runtime
```
4. Commit and push your changes 

<details>
    <summary>Git Commands </summary>

```bash
git add .github/workflows/dependency-review.yml
git commit -m "Add Dependency Review workflow"
git push origin main
```
</details>

5. Create a branch from the `main` branch

<details>
    <summary>Git Commands </summary>

  ```bash
git checkout -b feature-a
git push origin feature-a
```
</details>

6. Open a terminal in Codespaces and run `npm install json-web-token`
7. Commit and push the code 

<details>
    <summary>Git Commands </summary>

 ```bash
git add .
git commit -m "added json-web-token dependency"
git push
```
</details>

8. Raise a Pull Request to the `main` branch

#### Discussion Points
- Why is it important to manage open-source licenses in addition to vulnerabilities?
- What steps can teams take to ensure compliance with allowed licenses?
- What strategies can be implemented to minimize friction during reviews?
- How do you balance security requirements with the need to avoid workflow bottlenecks?
- When should you allow exceptions for flagged vulnerabilities?
  

### Lab 10 - Establish an Organization‑wide SECURITY.md Policy
#### Objective
Set up a default `SECURITY.md` in a centralized `.github` repository so all your organization’s repositories inherit a baseline security policy.

#### Steps
1. Create a new repository under your organization named `.github` (skip if it already exists).
2. Open the `.github` repo and click **Add file** > **Create new file**.
3. Name the file `SECURITY.md` (at the root of the repository).
4. Populate it with your organization’s security policy, for example:
   - **Reporting vulnerabilities**
   - **Supported versions**
   - **Security configurations applied**
   - **Contact & triage process**
5. Commit the file to the **main** branch.
6. To verify, navigate to any repository in your organization that does *not* have its own `SECURITY.md`. Under the **Code** tab you should see **Security policy** pointing to the default from `.github/SECURITY.md`.

<details>
  <summary>Optional: Example SECURITY.md template</summary>
 
  ```markdown
  # Security Policy

  ## Reporting a Vulnerability
  Please open an issue in this repository or email security@example.com.

  ## Supported Versions
  We support the latest minor releases of each major version.

  ## Security Configurations
  - Code scanning via CodeQL
  - Dependabot enabled

  ## Contact & Triage
  Security team: security@example.com
  ```

</details>

#### Discussion Points
- How does a centralized `SECURITY.md` help with security policy consistency?
- What are the advantages of having a default security policy for all repositories?
- When should a repository override this default with its own `SECURITY.md`? 
---

### Lab 11 - Identify Top Vulnerable Ecosystems and Repositories
#### Objective
Use GitHub’s Security dashboard to find critical alert hotspots in your most prominent package ecosystem.

#### Steps
1. On the Organization page navigate to the `Security` tab.
2. Use the `Overview` section and the `Detection` tabs to identify:
   - **Top vulnerable ecosystems**: List the top 3 ecosystems with the most open critical alerts.
   - **Repositories with most alerts**: List the top 5 repos with open critical alerts.

<details>
  <summary>Need a hand? Start with this filtering options</summary>
  
 ```markdown
  archived:false tool:dependabot dependabot.ecosystem:pip
  ```
</details>

#### Discussion Points
- How will you prioritize remediation across ecosystems?
- What processes ensure ongoing visibility of these metrics?
