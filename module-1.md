# Module 1 - Supply Chain Security

## Enablement  

### Lab 1 - Setting Up a Custom Security Configuration and Enabling Dependency Graph

In this lab, you will learn how to create and apply a custom security configuration to repositories in your organization. 

Follow the steps below to complete the exercise:

1. Click on your Organization's settings. In the `Security` section of the sidebar, select the `Code security` dropdown menu, then click `Configurations`.  You will be navigated to the `Code security configurations` page, click `New configuration` button.
<details>
  <summary> Animated Guide</summary>

![alt text](images/new-config.gif)

</details>

2. To help identify your custom security configuration and clarify its purpose, name your configuration and create a description. 

<details>
  <summary>Need Help? View Configuration Screenshot</summary>
  
![alt text](images/confignameanddesc.png)

</details>

3. For each configuration option select the following:
 - `GitHub Advanced Security Features`: Select `Include`.
 - `Dependency Graph`: Select `Enabled`.
 - All Other Settings: Select `Not set`.
 - In the `Policy` options, for `Use as default for newly created repositories`, select `All repositories`.
 - In the `Policy` options, for `Enforce Configuration`, select `Enforce`.

4. Click on `Save Configuration` button. Please confirm save if prompted.

  <details>
 <summary>Need Help? View Configuration Screenshot</summary>  
   
   ![alt text](images/configuration-options.png)
   
 </details>

5. The page will redirected to the `Configurations` page. Click on the `Apply to` dropdown and select `All repositories`. There will be a prompt for confirmation, select `Apply`.
   
<details>
  <summary>Animated Guide</summary>
  
**TO DO: GIF THIS WHEN TOTAL NUMBER OF REPOS CONFIRMED**

![alt text](images/applytoallrepos.png)
</details>

## Supply Chain Security - Know Your Environment

### Lab 2 - Dependency Graph

#### Exercise: Organization Level Dependencies 
1. On the Organization page, locate the `Insights` tab in the navigation bar at the top. Under the `Insights` sections, find and click on `Dependencies` from the left-hand menu. 
2. Review the licences used in the Organization.
3. Explore the relationship between dependencies and dependents.
<details>
  <summary> Animated Guide</summary>

![alt text](images/org-dependency-graph.gif)

</details>

#### Exercise: Repository Level Dependencies 
1. On the repository page, locate the `Insights` tab in the navigation bar at the top. Under the `Insights` sections, find and click on `Dependency Graph` from the left-hand menu. 
2. Carefully review the list of dependencies displayed and verify completeness. Look for any missing dependencies.
<details>
  <summary> Animated Guide</summary>

![alt text](images/repo-dependency-graph.gif)

</details>

### Lab 3 - Dependency Submission API 

#### Exercise: Dependency Submission Action

In this exercise we'll learn how to use the Dependency Submission Action to correctly populate the dependency graph.

1. Navigate to the `moshi` repository in your GitHub Organization
2. Notice the dependency graph only shows 3 dependencies. This is unusual for a project of its size. This happens because the repository uses Gradle for its build process, and Gradle resolves dependencies dynamically during build time.
3. Add the Dependency Submission Action. Fortunately, Gradle provides a GitHub Action that can generate and submit a dependency graph for Gradle projects.
  a. Navigate to the `.github/workflows` directory in your repository and create the `dependency-submission.yml` file
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

4. Commit and push the changes to the repository

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

### Lab 4 - Software Bill Of Materials (SBOM) Generation and Attestations

#### Exercise: 

A Software Bill of Materials (SBOM) is a comprehensive list of software components, dependencies, and versions within a project. Generating an SBOM helps improve supply chain security and enables transparency about the software used.

Methods to Retrieve SBOM:
- UI: You can download an SBOM directly via GitHub's UI under the Dependency graph or Security features.
- REST API: GitHub provides a REST API to retrieve SBOMs programmatically.
- GitHub Action: For automation, the `sbom-generator-action` uses GitHub's dependency graph to automatically build an SBOM in SPDX 2.3 format. It supports the same [ecosystems](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph) as the dependency graph. If you need support for a different set of formats, we recommend having a look at the [Microsoft SBOM Tool](https://github.com/microsoft/sbom-tool), or Anchore's [Syft](https://github.com/anchore/syft)

In this lab, we will use a GitHub Action to generate, upload, and attest the SBOM.

1. Navigate to the `mona-gallery`repository in your organization
2. Create a new workflow file named `generate-sbom.yml` in the `.github/workflows` directory. 
3. Add the following contents to the file to generate an SCOM using the [sbom-generator-action](https://github.com/advanced-security/sbom-generator-action): 

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
        uses: actions/upload-artifact@v3.1.0
        with:
          path: ${{ steps.sbom.outputs.fileName }}
          name: "SBOM"
```

5. SBOM attestation ensures the integrity, authenticity, and trustworthiness of a Software Bill of Materials by proving it was generated from a secure and verified process. Update the workflow to use the [attest-build-provenance action](https://github.com/actions/attest-build-provenance/tree/main) to create an attestation for our SBOM

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
