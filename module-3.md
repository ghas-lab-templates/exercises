# Module 3 - Code Scanning

---

## Lab 1 - Default Setup 

#### Objective
In this lab, you will learn how to configure code sacanning default setup at the Organization level.

Default setup for code scanning is the quickest, easiest, most low-maintenance way to enable code scanning for your repositories. Based on the code in your repository, 
default setup will automatically create a custom code scanning configuration. After enabling default setup, the code written in CodeQL-supported languages in your repository will be scanned. We will be using the Org that you have been given access to configure Default CodeQL scans at the organization level.

#### Steps

1. In GitHub, navigate to your **Organization** `Settings`.
2. Within the **Security** group in the sidebar, click `Advanced Security` > `Configurations`.
3. Navigate to your recently created config, `Code Security - Basic` and click on the `Apply to` drop down menu.
4. Note that we have the option of applying the configuration to:
    - **All repositories**
    - **All repositories without configurations**

5. For this exercise, we will instead select the specific repositories under our org by using the `Apply configurations` section, where we will select all repositories _EXCEPT_ the **mono** repository.
6. After selecting the other repositories, click the `Apply configuration` drop down menu and select the configuration you created in the previous lab, `Code Security - Basic`.
7. Finally click on `Apply` (this will start a CodeQL scan for all the eligible repositories under this org).

> [!IMPORTANT]
> Do not apply the security configuration to the **mono** repository.

<details>
  <summary> Animated Guide </summary>  
    
  ![create-branch](/images/code-scan-default-settings-5.gif)

</details>

## Lab 2 - Policies: Branch Ruleset to require code scanning results 

#### Objective 

Ensuring secure code is crucial in software development. Configuring branch protection rules to require passing code scanning results helps enforce security best practices and prevents vulnerabilities from being merged into the main branch.

You will complete this exercise using GitHub's web UI at the organization level.

#### Steps 

1. Navigate to the `Settings` tab at the `Organization` level.
2. In the left sidebar, select `Repository` and then click on `Rulesets`.
3. Click the green button labelled `New ruleset` and choose `New branch ruleset`.
4. Enter a clear name for your ruleset (e.g `Require CodeQL results on PR`).
5. Select `Enforcement status` to `Active`.
6. Under `Target repositories` select `All repositories`. 
7. Under `Target branches` select `Add target` and choose `Include default branch` from the dropdown menu. 
8. Under `Branch rules`, select`Require code scanning results`. Note the default settings (`Security alerts: High or higher` and `Alerts: Errors`).
9. Click the green `Create` button to finalise and activate your branch ruleset.

We will verify the new ruleset by creating a pull request to the default branch in the next lab.

   <details>
      <summary> Animated Guide </summary>  
        
  ![rulesets](/images/rulesets.gif)
  
   </details>

#### Discussion Points

1. Why would setting up branch rulesets for code scanning results be beneficial for an organization? 
2. How does enforcing rules at the organization level help maintain security consistency?
3. What potential issues could arise from incorrectly configured branch rulesets?
4. How could you adapt this ruleset for different types of repositories within your organization?
5. What additional rules or security measures might complement required code scans?

## Lab 3 - Autofix on PR 

> [!NOTE]
> Open the `mona-gallery` repository in your organization.

#### Objective

Input sanitization is a fundamental security practice to prevent SQL injection attacks. Let's add a sanitize method in javascript to help in mitigating against injection attack vectors.

You can solve this exercise using either a local IDE or the UI. Local IDEs are preferable as this is what a developer would use under normal circumstances. However, the UI will suffice for such a simple fix.

#### Steps

1. If using an IDE, create a branch called `sql-injection-fix` and push it to the remote repo.

    <details>
      <summary> Hint </summary>
      Run the command:
        
      ```
        $ git checkout -b sql-injection-fix
        $ git push -u origin sql-injection-fix
      ```
     </details>

    <details>
      <summary> Animated Guide </summary>  
        
      ![create-branch](/images/create-branch.gif)
  
    </details>
     

2. Add the following sanitization function on **line 233** of `frontend/src/components/Gallery.vue`

    ```js
    function sanitizeInput(input) {
        if (input == null) {
            return "";
        }
        //escape all occurances of apostrophe
        input = input.replace("'", "''");
    
        return input;
    }
    ```

3. Call the sanization function from the Update function by placing the following call on **line 347** of `/frontend/components/Gallery.vue`

    ```js
        artItem.description = sanitizeInput(artItem.description)
        artItem.title = sanitizeInput(artItem.title)
    ```

4. Commit and push the code (if using the UI, set the branch name to `sql-injection-fix` and propose the changes).

    <details>
       <summary> Animated Guide </summary>  
        
       ![sanitization](/images/sanitise-method.gif)
       
    </details>

5. Create a pull request to the `main` branch. Wait for the scans to complete and you should see a CodeQL javascript alert in your pull request. Oh no! There is a vulnerability in our sanitization! Luckily we have Copilot Autofix. Review the fix generated by Autofix, and if satisfied with it, accept it. This will retrigger a CodeQL scan and the alert should be resolved.

> [!IMPORTANT]
> Don't merge the pull request just yet, we will add to it in the next lab.

## Lab 4 - Autofix on Open Alerts

#### Objective 
In this lab, we will address two CodeQL alerts related to SQL injection vulnerabilities in our Go code.

#### Steps

1. Navigate to the `Security` tab in the `mona-gallery` repository.
2. Click on the `Code scanning` tab.
3. Use the `Rule` filter and select `Database query built from user-controlled sources`.
4. Navigate to each SQL injection alert and click  the `Generate fix` button
5. Once the fix is generated, carefully review it to ensure accuracy. AI-generated suggestions can vary, so verifying each proposed solution is crucial.
6. Click the dropdown arrow on the green `Commit to new branch` button and select `Commit to branch`.
7. Choose the `sql-injection-fix` branch from the dropdown options, and select `Open a pull request`. This will append your changes to the existing PR.

Proceed with these steps until both alerts are resolved.

<details>
   <summary> Animated Guide  </summary>  
    
   ![sanitization](/images/generate-fix.gif)
   
</details>


## Lab 5 - Advanced Configuration with custom build command

> [!NOTE]
> Open the `JavaVulnerableLab` repository in your organization.

### Objective
In this lab, you will learn how to configure a CodeQL scan with Advanced Setup (an Actions workflow file). We will be defining a custom CodeQL scanning yml file, but with a custom build command to build the application before scanning the application for vulnerabilities. This will showcase how custom build commands can be used to scan compiled languages in CodeQL.

#### Steps

1. Create the file `.github/workflows/codeql.yml` by using the Actions marketplace template for CodeQL, which can be found by navigating to the `Actions` tab.
   <details>
     <summary>Hint</summary>

    1. Navigate to `Actions` tab
    2. If you've already run a workflow in this repository before, select the `New workflow` button. Otherwise, skip to step 3.
    3. Search for `CodeQL` in the search bar
    4. From the list of results, select `Configure` under `CodeQL Analysis`
    5. This will open up the Actions editor with a pre-defined template populated to run the CodeQL scan

   </details>

   <details>
     <summary>Solution</summary>

     ```yaml
        # For most projects, this workflow file will not need changing; you simply need
        # to commit it to your repository.
        #
        # You may wish to alter this file to override the set of languages analyzed,
        # or to provide custom queries or build logic.
        #
        # ******** NOTE ********
        # We have attempted to detect the languages in your repository. Please check
        # the `language` matrix defined below to confirm you have the correct set of
        # supported CodeQL languages.
        #
        name: "CodeQL Advanced"

    on:
      push:
        branches: [ "main" ]
      pull_request:
        branches: [ "main" ]
    
    jobs:
      analyze:
        name: Analyze (${{ matrix.language }})
        runs-on: ${{ (matrix.language == 'swift' && 'macos-latest') || 'ubuntu-latest' }}
        permissions:
          security-events: write
          packages: read
          actions: read
          contents: read
    
        strategy:
          fail-fast: false
          matrix:
            include:
            - language: java-kotlin
              build-mode: none # This mode only analyzes Java. Set this to 'autobuild' or 'manual' to analyze Kotlin too.
        # CodeQL supports the following values keywords for 'language': 'c-cpp', 'csharp', 'go', 'java-kotlin', 'javascript-typescript', 'python', 'ruby', 'swift'
        # Use `c-cpp` to analyze code written in C, C++ or both
        # Use 'java-kotlin' to analyze code written in Java, Kotlin or both
        # Use 'javascript-typescript' to analyze code written in JavaScript, TypeScript or both
        # To learn more about changing the languages that are analyzed or customizing the build mode for your analysis,
        # see https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning.
        # If you are analyzing a compiled language, you can modify the 'build-mode' for that language to customize how
        # your codebase is analyzed, see https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages
        
        steps:
        - name: Checkout repository
          uses: actions/checkout@v4
    
        - name: Initialize CodeQL
          uses: github/codeql-action/init@v3
          with:
            languages: ${{ matrix.language }}
            build-mode: ${{ matrix.build-mode }}

         # If the analyze step fails for one of the languages you are analyzing with
         # "We were unable to automatically build your code", modify the matrix above
         # to set the build mode to "manual" for that language. Then modify this step
         # to build your code.
         # ℹ️ Command-line programs to run using the OS shell.
         # 📚 See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun
        - if: matrix.build-mode == 'manual'
          shell: bash
          run: |
            echo 'If you are using a "manual" build mode for one or more of the' \
              'languages you are analyzing, replace this with the commands to build' \
              'your code, for example:'
            echo '  make bootstrap'
            echo '  make release'
            exit 1
    
        - name: Perform CodeQL Analysis
          uses: github/codeql-action/analyze@v3
          with:
            category: "/language:${{matrix.language}}"
     ```
   </details>

2.  If you analyse the yml file, you can see that by default the `build-mode` configured is `none` (buildless scan)
3.  As per the comments in the auto generated yml file, if we have to run our custom build command for this java application, we have to change the `build-mode` to `manual`
4.  Can you suggest what changes you have to make to the CodeQL yml file to run a custom build command before a scan is triggered with codeQL?

<details>
     <summary>Hint</summary>

1. Change the `build-mode` from `none` to `manual`
2. In the `Initialize CodeQL` step, under the `if` condition which corresponds to the `manual` build step, comment out the `echo` commands and the `exit 1` command
3. Replace these with the actual java build command. In this case, since it is a maven application, we would use the below command to build the source code:

        `mvn clean install`
 </details>

 <details>
     <summary> Final Solution</summary>

```yaml
        # For most projects, this workflow file will not need changing; you simply need
        # to commit it to your repository.
        #
        # You may wish to alter this file to override the set of languages analyzed,
        # or to provide custom queries or build logic.
        #
        # ******** NOTE ********
        # We have attempted to detect the languages in your repository. Please check
        # the `language` matrix defined below to confirm you have the correct set of
        # supported CodeQL languages.
        #
        name: "CodeQL Advanced"

    on:
      push:
        branches: [ "main" ]
      pull_request:
        branches: [ "main" ]
    
    jobs:
      analyze:
        name: Analyze (${{ matrix.language }})
        runs-on: ${{ (matrix.language == 'swift' && 'macos-latest') || 'ubuntu-latest' }}
        permissions:
          security-events: write
          packages: read
          actions: read
          contents: read
    
        strategy:
          fail-fast: false
          matrix:
            include:
            - language: java-kotlin
              build-mode: manual # This mode only analyzes Java. Set this to 'autobuild' or 'manual' to analyze Kotlin too.
        # CodeQL supports the following values keywords for 'language': 'c-cpp', 'csharp', 'go', 'java-kotlin', 'javascript-typescript', 'python', 'ruby', 'swift'
        # Use `c-cpp` to analyze code written in C, C++ or both
        # Use 'java-kotlin' to analyze code written in Java, Kotlin or both
        # Use 'javascript-typescript' to analyze code written in JavaScript, TypeScript or both
        # To learn more about changing the languages that are analyzed or customizing the build mode for your analysis,
        # see https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning.
        # If you are analyzing a compiled language, you can modify the 'build-mode' for that language to customize how
        # your codebase is analyzed, see https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages
        
        steps:
        - name: Checkout repository
          uses: actions/checkout@v4
    
        - name: Initialize CodeQL
          uses: github/codeql-action/init@v3
          with:
            languages: ${{ matrix.language }}
            build-mode: ${{ matrix.build-mode }}

         # If the analyze step fails for one of the languages you are analyzing with
         # "We were unable to automatically build your code", modify the matrix above
         # to set the build mode to "manual" for that language. Then modify this step
         # to build your code.
         # ℹ️ Command-line programs to run using the OS shell.
         # 📚 See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun
        - if: matrix.build-mode == 'manual'
          shell: bash
          run: |
            # echo 'If you are using a "manual" build mode for one or more of the' \
            #   'languages you are analyzing, replace this with the commands to build' \
            #   'your code, for example:'

            mvn clean install
    
        - name: Perform CodeQL Analysis
          uses: github/codeql-action/analyze@v3
          with:
            category: "/language:${{matrix.language}}"
```
</details>

5. Commit the changes to a new branch, for example `codeql-advanced-setup`, and push it to the repository.

## Lab 6 - Security Campaigns

#### Objective
Security campaigns are a way to group alerts and prioritize them with developers, so you can focus on remediation of those select vulnerabilities. In this lab we will create a security campaign for identified security alerts in the default branch and see how the Campaigns feature in GHAS will help encourage focussed remediation effort from development teams. With this feature you can fix security alerts at scale by creating security campaigns and collaborating with developers to burn down your security backlog.

#### Steps

For this exercise, we will use the Org level Security tab for creating `Campaigns`.

 1. In GitHub, navigate to your **Organization** `Security` tab.
 2. Under the `Security` tab, click on the `Campaigns` menu in the left-hand pane.
 3. Click on the `Create campaign` button.
 4. Select `From template`
 5. In the subsequent menu, review the existing Campaign templates:
    1. Critial CodeQL Alerts
    2. Mitre top 10 KEV
    3. SQL Injection
    4. Cross-site Scripting

<details>
  <summary> Animated Guide</summary>

![alt text](images/Security-Campaign-01.gif)

</details>

**Challenge:** Rather than using a template, can you create a custom campaign for all the open vulnerabilities of severity `critical` and have `autofix` support?

 <details>
  <summary> Solution </summary>

![alt text](images/Security-Campaign-02.gif)

</details>  

## Lab 7 - Code Scanning for monorepos

> [!NOTE]
> Open the `mono` repository in your organization.

#### Objective

In this lab, you will learn how to configure CodeQL for efficient monorepo scanning.

Monorepos are large in size, which often leads to longer scan times that can delay PR workflows. To optimize performance, teams often choose to split scans, focusing only on the relevant parts of the repository affected by code changes. The feasibility of splitting scans depends on the architecture of the monorepo.

This lab demonstrates the concept of monorepo scanning with CodeQL. However, the way it is ultimately divided will depend on factors such as the bottlenecks in the analysis, the tech stack, and the architecture.

Due to time and size constraints of real-world monorepos, `mono` is a smaller example, but its setup follows the same principles as a full-scale monorepo. This repository is a copy of the mona-gallery template and consists of multiple independent application tiers, each organized into separate directories:

- Directory: auth
    - Language: go
    - Build Mode: autobuild
- Directory: frontend
    - Language: javascript
    - Build Mode: none
- Directory: storage
    - Language:java
    - Build Mode: none
- Directory: auth-ext    
    - Language:python
    - Build Mode: none
- Directory: gallery
    - Language go
    - Build Mode: autobuild
- Directory: .github
    - Language: actions
    - Build Mode: none

These are the only directories within the `mono` repo that we're interested in scanning. Any changes outside these directories should be skipped.

#### Steps

Earlier, we chose not to apply our security configuration to the `mono` repository. Now, we will create a custom CodeQL workflow that scans only the relevant directories in the monorepo. This will help us understand how to efficiently manage code scanning in large repositories.

1. Navigate to the `Settings` tab of the `mono` repository.
2. In the left sidebar, select `Advanced Security`.
3. Enable `GitHub Advanced Security` for the repository (if it's not yet enabled).
4. Next to `CodeQL analysis`, click the `Set up` dropdown, then select `Advanced`.
5. You'll be directed to save a workflow template in the `.github/workflows` directory. You will edit this workflow in the following steps.
6. The workflow you are creating will contain three distinct jobs:
     - **detect_changes**: Identifies modified areas of your monorepo.
     - **analyze**: Performs the actual CodeQL analysis based on the detected changes.
     - **process_sarif**: Processes directories that have not changed. 
     
     Your task is to:
     1. Understand the YAML structure in the template below.
     2. Familiarize yourself with the GitHub Actions structure, which includes jobs, steps, and workflow triggers. You can read more about GitHub Actions [here](https://docs.github.com/en/actions).
     3. Navigate to `.github/scripts` and review the scripts:
      - Understand the role of `process.awk` in mapping file changes to directories and languages.
      - Verify how JSON outputs for changes (`matrix`) and unchanged directories (`matrix_no_changes`) are generated.

        <details>
          <summary>Explanation</summary>
          
               
          This awk script processes a configuration file (`cfg_for_dir.txt`) that identifies the programming language and build mode for each directory. It then checks which directories have changes and which do not, and outputs this information in JSON format.
                     
          Here is a step-by-step explanation of the script:
                     
          `BEGIN` Block:
        
          Reads the `cfg_for_dir.txt` file line by line.
          Each line is split into fields based on the semicolon delimiter.
          Populates the `cfg_for_dir` associative array with the directory path as the key, and another associative array as the value, which contains the language and build mode for that directory.
                     
          `Main` Block:
        
          For each record processed, it checks if the directory (the first field) is in `cfg_for_dir`.
          If the directory is not yet in the `dirs` array, it adds an entry to the dirs array with JSON-formatted information about the directory, language, and build mode.
          Also, it iterates through all keys in `cfg_for_dir` and checks if they are not in `dirs`. If they are not, it adds them to the `no_changes` array with similar JSON-formatted information.
                
          `END` Block:
        
          Outputs the contents of `dirs` and `no_changes` arrays in JSON format.
          The `changes` array contains directories where files have changed, while the `no_changes` array contains directories where no files have changed.
          The final output is a JSON object that lists directories with changes and directories without changes, each with their corresponding language and build mode. This can be used for further processing, such as code 
          analysis or build orchestration.

        </details>

   **Monorepo CodeQL Template**
    ```yaml

          name: "CodeQL Monorepo Analysis"
    
          on:
            push:
              branches: [ "main" ]
            pull_request:
              branches: [ "main" ]
            schedule:
              - cron: '19 1 * * 1'
    
          jobs:
            detect_changes:
              runs-on: ubuntu-latest
              permissions:
                actions: read
                contents: read
              outputs:
                matrix: ${{ steps.detect_changes.outputs.matrix }}
                matrix_no_changes: ${{ steps.detect_changes.outputs.matrix_no_changes }}
              steps:
                - name: Checkout repository
                  uses: actions/checkout@v4
                  with:
                    fetch-depth: 2        
                - name: Find changed directories and map to directories and languages
                  id: detect_changes
                  run: |
                    # TODO: Implement detection logic
    
            analyze:
              name: Analyze (${{ matrix.language }})
              needs: detect_changes
              runs-on: ubuntu-latest
              permissions:
                security-events: write
                packages: read
                actions: read
                contents: read
              strategy:
                fail-fast: true
                matrix:
                  include: ${{ fromJson(needs.detect_changes.outputs.matrix) }}
    
              steps:
                - name: Checkout repository
                  uses: actions/checkout@v4
                  with:
                    sparse-checkout: |
                      ${{ matrix.directory }}
                    sparse-checkout-cone-mode: false
                # TODO: Implement CodeQL analysis steps
    
            process_sarif:
              name: Process SARIF
              needs: [detect_changes, analyze]
              runs-on: ubuntu-latest
              permissions:
                # required for all workflows
                security-events: write
                # required to fetch internal or private CodeQL packs
                packages: read
                # only required for workflows in private repositories
                actions: read
                contents: read
              strategy:
                fail-fast: true
                matrix: 
                  include:  # TODO: Implement what needs to be passed into the matrix
    
              steps:
              - name: Checkout repository
                uses: actions/checkout@v4
                with:
                  sparse-checkout: |
                     # TODO: Implement what needs to be checked out
                  sparse-checkout-cone-mode: false
              
               # TODO: Implement processing unchanged files 
                    

    ```    


7. Update the action template to detect changes on both `push` and `pull_request` events. This should be implemented in the `detect_changes` job within the `# TODO: Implement detection logic` section.

   <details>
     <summary>Hint</summary>

     - To detect changes on a pull request, use:
       
          `git diff --name-only ${{ github.event.pull_request.base.sha }} ${{ github.event.pull_request.head.sha }}`
       
     -  To detect changes on a push use:
       
          `git diff --name-only HEAD^ HEAD`
        
     -  Ensure the workflow fetches enough history by setting `fetch-depth: 2` on checkout.

   </details>

   <details>
     <summary>Solution</summary>

    ```yaml

      name: "CodeQL Monorepo Analysis"

      on:
        push:
          branches: [ "main" ]
        pull_request:
          branches: [ "main" ]
        schedule:
          - cron: '19 1 * * 1'

      jobs:
        detect_changes:
          runs-on: ubuntu-latest
          permissions:
            actions: read
            contents: read
          outputs:
            matrix: ${{ steps.detect_changes.outputs.matrix }}
            matrix_no_changes: ${{ steps.detect_changes.outputs.matrix_no_changes }}
          steps:
            - name: Checkout repository
              uses: actions/checkout@v4
              with:
                fetch-depth: 2        
            - name: Find changed directories and map to directories and languages
              id: detect_changes
              run: |
                cd .github/scripts
                if [[ ${{ github.event_name }} == 'pull_request' ]]; then
                DIFF=$(git diff --name-only ${{ github.event.pull_request.base.sha }} ${{ github.event.pull_request.head.sha }} | awk -f process.awk ) 
                CHANGES=$(echo "$DIFF" | jq -c '.changes')
                NO_CHANGES=$(echo "$DIFF" | jq -c '.no_changes')
              
                else
                  # For push events, compare with the previous commit
                  DIFF=$(git diff --name-only HEAD^ HEAD | awk -f process.awk)
                  CHANGES=$(echo "$DIFF" | jq -c '.changes')
                  NO_CHANGES=$(echo "$DIFF" | jq -c '.no_changes')
                fi

                # Store in output and also in a variable for debugging
                echo "matrix=$CHANGES" >> $GITHUB_OUTPUT
                echo "matrix_no_changes=$NO_CHANGES" >> $GITHUB_OUTPUT
          
                # Print the changes for debugging
                echo "Changes found: $CHANGES" 
                echo "No changes found: $NO_CHANGES"    

        ....
    ```   
     
   </details>

  8.  Update the action file to perform CodeQL analysis. Modify the job `analyze` to include CodeQL analysis for different components of the project using the category property.

       <details>
          <summary>Hint</summary>
          
    
        1. Initialize CodeQL using:
           
         ```yaml
            - name: Initialize CodeQL
              uses: github/codeql-action/init@v3
         ```
                  
        2. Perform CodeQL analysis using:
            
         ```yaml
            - name: Perform CodeQL Analysis
              uses: github/codeql-action/analyze@v3
         ```
    
        3. Use the `category` property to analyze different components separately, such as:
                `"/language:${{matrix.language}}/app:${{matrix.directory}}"`
    
       </details>    
    
    
       <details>
          <summary>Solution</summary>
              
         ```yaml
    
                analyze:
                  name: Analyze (${{ matrix.language }})
                  needs: detect_changes
                  runs-on: 'ubuntu-latest' 
                  permissions:
                    # required for all workflows
                    security-events: write
                    # required to fetch internal or private CodeQL packs
                    packages: read
                    # only required for workflows in private repositories
                    actions: read
                    contents: read
                  strategy:
                    fail-fast: true
                    matrix: 
                      include: ${{ fromJson(needs.detect_changes.outputs.matrix) }}
    
                  steps:
                      - name: Checkout repository
                        uses: actions/checkout@v4
                        with:
                          sparse-checkout: |
                            ${{ matrix.directory }}
                            .github/scripts/empty.sarif
                          sparse-checkout-cone-mode: false       
                      - name: Initialize CodeQL
                        uses: github/codeql-action/init@v3
                        with:
                            languages: ${{ matrix.language }}
                            build-mode: ${{ matrix.build_mode }}
                      - name: Perform CodeQL Analysis
                        uses: github/codeql-action/analyze@v3
                        with:
                          category: "/language:${{matrix.language}}/app:${{matrix.directory}}"

         ```
       </details>

   9. Modify the action file to handle the `process_sarif` job, ensuring that all required checks pass by submitting an empty SARIF report for unchanged components.

      - If a push rule requires all checks to pass before merging, and CodeQL does not analyze certain parts of the monorepo, those checks will remain in a neutral state and merging will be blocked.
      
      - To address this, you need to submit can empty SARIF file (available in `.github/scripts/empty.sarif`) for any unmodified components to mark them as successfully analyzed.



      <details>
          <summary>Solution</summary>
     
        ```yaml        
              process_sarif:
                name: Process SARIF
                needs: [detect_changes, analyze]
                runs-on: ubuntu-latest
                permissions:
                  # required for all workflows
                  security-events: write
                  # required to fetch internal or private CodeQL packs
                  packages: read
                  # only required for workflows in private repositories
                  actions: read
                  contents: read
                strategy:
                  fail-fast: true
                  matrix: 
                    include: ${{ fromJson(needs.detect_changes.outputs.matrix_no_changes) }}

                steps:
                - name: Checkout repository
                  uses: actions/checkout@v4
                  with:
                    sparse-checkout: |
                      .github/scripts/empty.sarif
                    sparse-checkout-cone-mode: false
                
                - name: Process SARIF
                  uses: github/codeql-action/upload-sarif@v3
                  with:
                      sarif_file: .github/scripts/empty.sarif
                      category: "/language:${{matrix.language}}/app:${{matrix.directory}}"                      
         ```

       </details>


#### Final Solution

<details>
<summary>Solution</summary>

```yaml
   # For most projects, this workflow file will not need changing; you simply need
   # to commit it to your repository.
   #
   # You may wish to alter this file to override the set of languages analyzed,
   # or to provide custom queries or build logic.
   #
   # ******** NOTE ********
   # We have attempted to detect the languages in your repository. Please check
   # the `language` matrix defined below to confirm you have the correct set of
   # supported CodeQL languages.
   #
   name: "CodeQL Monorepo Analysis"
   on:
     push:
       branches: [ "main" ]
     pull_request:
       branches: [ "main" ]
     schedule:
       - cron: '19 1 * * 1'
   jobs:
     detect_changes:
       runs-on: ubuntu-latest
       permissions:
         # only required for workflows in private repositories
         actions: read
         contents: read
       outputs:
         matrix: ${{ steps.detect_changes.outputs.matrix }}
         matrix_no_changes: ${{ steps.detect_changes.outputs.matrix_no_changes }}
       steps:
         - name: Checkout repository
           uses: actions/checkout@v4
           with:
             fetch-depth: 2
         - name: Find changed directories and map to directories and languages
           id: detect_changes
           run: |
             cd .github/scripts
             if [[ ${{ github.event_name }} == 'pull_request' ]]; then
             DIFF=$(git diff --name-only ${{ github.event.pull_request.base.sha }} ${{ github.event.pull_request.head.sha }} | awk -f process.awk )
             CHANGES=$(echo "$DIFF" | jq -c '.changes')
             NO_CHANGES=$(echo "$DIFF" | jq -c '.no_changes')
             else
               # For push events, compare with the previous commit
               DIFF=$(git diff --name-only HEAD^ HEAD | awk -f process.awk)
               CHANGES=$(echo "$DIFF" | jq -c '.changes')
               NO_CHANGES=$(echo "$DIFF" | jq -c '.no_changes')
             fi
             # Store in output and also in a variable for debugging
             echo "matrix=$CHANGES" >> $GITHUB_OUTPUT
             echo "matrix_no_changes=$NO_CHANGES" >> $GITHUB_OUTPUT
             # Print the changes for debugging
             echo "Changes found: $CHANGES"
             echo "No changes found: $NO_CHANGES"
     analyze:
       name: Analyze (${{ matrix.language }})
       needs: detect_changes
       runs-on: 'ubuntu-latest'
       permissions:
         # required for all workflows
         security-events: write
         # required to fetch internal or private CodeQL packs
         packages: read
         # only required for workflows in private repositories
         actions: read
         contents: read
       strategy:
         fail-fast: true
         matrix:
           include: ${{ fromJson(needs.detect_changes.outputs.matrix) }}
       steps:
           - name: Checkout repository
             uses: actions/checkout@v4
             with:
               sparse-checkout: |
                 ${{ matrix.directory }}
                 .github/scripts/empty.sarif
               sparse-checkout-cone-mode: false
           - name: Initialize CodeQL
             uses: github/codeql-action/init@v3
             with:
                 languages: ${{ matrix.language }}
                 build-mode: ${{ matrix.build_mode }}
           - name: Perform CodeQL Analysis
             uses: github/codeql-action/analyze@v3
             with:
              category: "/language:${{matrix.language}}/app:${{matrix.directory}}"
     process_sarif:
         name: Process SARIF
         needs: [detect_changes, analyze]
         runs-on: ubuntu-latest
         permissions:
           # required for all workflows
           security-events: write
           # required to fetch internal or private CodeQL packs
           packages: read
           # only required for workflows in private repositories
           actions: read
           contents: read
         strategy:
           fail-fast: true
           matrix:
             include: ${{ fromJson(needs.detect_changes.outputs.matrix_no_changes) }}
         steps:
         - name: Checkout repository
           uses: actions/checkout@v4
           with:
             sparse-checkout: |
               .github/scripts/empty.sarif
             sparse-checkout-cone-mode: false
         - name: Process SARIF
           uses: github/codeql-action/upload-sarif@v3
           with:
               sarif_file: .github/scripts/empty.sarif
               category: "/language:${{matrix.language}}/app:${{matrix.directory}}"
```
  </details>

#### Discussion Points

- Why is efficient scanning critical in large monorepos? How do lengthy scan times impact developer productivity and pull request workflows?

- What factors determine whether splitting monorepo scans is feasible? How can a monorepo's architecture influence the ease or complexity of implementing split scans?

- Why is it necessary to handle categories that have not changed by submitting empty SARIF reports?


## Lab 8 - Running Code Scanning with 3rd Party Scanners

> [!NOTE]
> Open the `terragoat-iac` repository in your organization.

#### Objective
The objective of this lab is to showcase how 3rd party code scanning tools can be integrated with GitHub Code Scannig (using Actions or 3rd party CI/CD). While CodeQL is a robust, built-in solution, we also have the flexibility to integrate other tools for a customized approach. By configuring code scanning with 3rd party Actions, we can incorporate tools like KICS (IaC code scanning), SonarQube (code quality checks), Trivy (container scanning), etc. These tools can produce results as SARIF files, which can be uploaded to GitHub Code Scanning to display alerts alongside GitHub's CodeQL scans, simplifying your code review process. This flexibility is ideal for teams already using external analysis tools, allowing all findings to be centralized in GitHub's Security tab for easier management.

In this lab we will be using KICS Infrastructure as Code scanner to scan terraform files in a repository.

#### Steps
1. We will be using [KICS IaC scanner](https://github.com/marketplace/actions/kics-github-action) to run the scan.
2. Familiarise yourself with the KICS GitHub Action and the various configuration options that the tool provides.
3. Now navigate to the `.github/workflows` folder and create a new `kics-scan.yml` file.
4. Referencing the KICS Gitub Action marketplace documentation, can you create the config that runs a KICS scan on the `terragoat-iac` repo, creates a SARIF file of the results, and then uploads the SARIF file to GitHub Security Overview dashboard using the `upload-sarif` action.
 

   <details>
   <summary>Solution</summary>

   ```yaml
   name: Scan with KICS and upload SARIF

   on:
    push:
       branches: [ "main" ]
    pull_request:
       branches: [ "main" ]

   permissions:
    contents: read
    actions: read
    security-events: write
    
   jobs:
     kics-job:
       runs-on: ubuntu-latest
       name: kics-action
       steps:
         - name: Checkout repo
           uses: actions/checkout@v4
         - name: Mkdir results-dir
           # make sure results dir is created
           run: mkdir -p results-dir
         - name: Run KICS Scan with SARIF result
           uses: checkmarx/kics-github-action@v2.1.7
           with:
             path: 'terraform'
             # when provided with a directory on output_path
             # it will generate the specified reports file named 'results.{extension}'
             # in this example it will generate:
             # - results-dir/results.json
             # - results-dir/results.sarif
             ignore_on_exit: results
             output_path: results-dir
             platform_type: terraform
             output_formats: 'json,sarif'
     
         - name: Show results
           run: |
             cat results-dir/results.sarif
             cat results-dir/results.json
         - name: Upload SARIF file
           uses: github/codeql-action/upload-sarif@v3
           with:
             sarif_file: results-dir/results.sarif
   ```
   
   </details>

5. Post scan completion, you can navigate to the `Security` tab in the GitHub UI and under `Code scanning` in the dashboard you can see all the IaC vulnerabilities identified by KICS scanner.

#### Discussion Points

- What 3rd party scanners are your teams are already using?

- Are there any challenges that dev teams are facing in terms of vulnerability analysis with multiple tools and dashboards?

