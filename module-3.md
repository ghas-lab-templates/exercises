# Module 3 - Code Scanning

---

## Lab 1 - Default CodeQL 

#### Objective
In this lab, you will learn how to configure the code sacanning default setup at the repository level. The default CodeQL scan can also be enforced at the Org level.

Default setup for code scanning is the quickest, easiest, most low-maintenance way to enable code scanning for your repository. Based on the code in your repository, 
default setup will automatically create a custom code scanning configuration. After enabling default setup, the code written in CodeQL-supported languages in your repository will be scanned. We will use the mona-gallery repository to initiate the default codeql scan for the repository

#### Steps

1. Click on your Repository settings. In the `Security` section of the sidebar, click the `Code security` menu on the left hand pane
2. Navigate to the `Code scanning` section in this page where you will see the CodeQL analysis option with the set up allowing the below scan config options
    1. Default
    2. Advanced
3. Click on the `Default` setup
4. The subsequent pop up window, you will see that that the Languages is automatically populated along with the Query suites, Runner type and Scan events on which the CodeQL default scan will trigger. And finally the schedule of scan
   
<details>
  <summary> Animated Guide</summary>

![alt text](images/code-scan-default-settings-1.gif)

</details>

5. If you wish to modify the scan parameters of the `Default scan` you can click on `Edit` button on this screen
6. In this Edit screen of Default Scanning, you can select the languages that you want to scan for (if the repository has multiple programming languages). By default all the programming languages that CodeQL supports will be selected
7. You can also select the runner type. For this exercise we will use the `Standard GitHub runner`
8. Next select the `Query suit`
9. Also there is an option to select `Threat model` to define and refine additional sources
10. Finally we have the `Scan events` which will define the event on which the scan will be triggered and also the frequency of scan (weekly)

<details>
  <summary> Animated Guide</summary>

![alt text](images/code-scan-default-settings-2.gif)

</details>

11. As the final step once you have validated all the config iptions in the `Default scan`, you can go ahead and click on the `Enable CdeQL` button
12. This will start the CodeQL analysis using the GitHub Hosted Runners. You can validate the scan progress in the `Actions` tab
13. Once the scan is completed for all the listed technologies, you can navigate to the `Security` tab and then under `Code scanning` you can see all the CodeQL vulnerabilities

<details>
  <summary> Animated Guide</summary>

![alt text](images/code-scan-default-settings-3.gif)

</details>

## Lab 2 - Policies: Branch Ruleset to require code scanning results 

#### Objective 

Ensuring secure code is crucial in software development. Configuring branch protection rules to require passing code scanning results helps enforce security best practices and prevents vulnerabilities from being merged into the main branch.

You will complete this exercise using GitHub's web UI at the organization level.

#### Steps 

1. Navigate to the `Settings` tab at the `Organization` level.
2. In the left sidebar, select `Repository` and then click on `Rulesets`.
3. Click the green button labelled `New ruleset` and choose `New branch ruleset`.
4. Enter a clear name for your ruleset (e.g Require CodeQL results on PR).
5. Select `Enforcement status` to `Active`.
6. Under `Target repositories` select `All repositories`. 
7. Under `Target branches` select `Add target` and choose `Include default branch` from the dropdown menu. 
8. Under `Branch rules`, select`Require code scanning results`. Keen the default settings (`Security alers:High or higher` and `Alerts:Errors`).
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
5. What additional rules or security measures might complement required code s
     

## Lab 3 - Autofix on PR 

#### Objective

Input sanitization is a fundamental security practice to prevent SQL injection attacks. Let's add a sanitize method in the javascript to help in mitigating against injection attack vectors.

You can solve this exercise using either the codespaces or the UI. Codespaces is preferable as this is what a developer would use under normal circumstances. However, if codespaces is not loading for you please use the UI. 

#### Steps

1. Create a branch called `sql-injection-fix` and push it to the remote repo.

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
     

2. Add the following sanitization function on **line 233** of `frontend/components/Gallery.vue`

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

4. Commit and push the code

    <details>
       <summary> Animated Guide  </summary>  
        
       ![sanitization](/images/sanitise-method.gif)
       
    </details>

5. Raise a pull request to the `main` branch. Wait for the scans to complete and you should see a CodeQL javascript alert in your pull request. Oh no! There is a vulnerability in our vulnerability! Lucky we have autofix. Review the fix generated by autofix and if satisfied with it accept it. 

## Lab 3 - Autofix on Open Alerts

#### Objective 
In this lab, we will address two CodeQL alerts related to SQL Injection vulnerabilities in our Go code.

#### Steps

1. Navigate to each SQL Injection alert and click  the `Generate Fix` button
2. Once the fix is generated, carefully review it to ensure accuracy. AI-generated suggestions can vary, so verifying each proposed solution is crucial 
3. Click the dropdown arrow on the green `Commit to new branch` button and select `Commit to branch`.
4. Choose the `sql-injection-fix` branch from the dropdown options, and select `Open a new pull request`. This will append your changes to the existing PR.

Proceed with these steps until both alerts are resolved.

<details>
   <summary> Animated Guide  </summary>  
    
   ![sanitization](/images/generate-fix.gif)
   
</details>


## Lab 4 - Advanced Configuration with custom build command

### Objective
In this lab, you will learn how to configure CodeQL scan with an advanced setup (Actions based). We will be defining a custom CodeQL scanning yml file, but with a custom build command to build the application before scanning the application for vulnerabilities. The idea is to showcase how custom build commands can be used to scan compiled languages in CodeQL. 

For this Lab, we will be using the `JavaVulnerableLab` repository

#### Steps

1. Create the file `.github/codeql/codeql-config.yml`
   <details>
     <summary>Hint</summary>

    a. You can navigate to `Security` tab and then navigate to the `Code scanning` menu under the `Vulnerability Alerts` list
    b. Then click on `Add tool`
    c. From the list of Security tools available, select the `CodeQL Analysis` and click `Configure`
    d. This would open up the Actions editior with a pre defined template populated to run the CodeQL scan

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

2.  If you analyse the yml file, you can see that by default the `build-mode` configured is `none` (build less scan)
3.  As per the comments in the auto generated yml file, if for some reason, we have to issue our custom build command for this java application, we have to change the `build-mode` to `manual`
4.  Can you suggest what changes you have to make to the CodeQL yml file to issue a custom build command before a scan is triggered with codeQL ?

<details>
     <summary>Hint</summary>

a. Change the `build-mode` to manual from `none`
b. In the `Initialize CodeQL` step, under the `if` condition which corresponds to the `manual` build step, comment out the `echo` commands and the `exit 1` command
c. Replace these with the actual java build command. In this case, since it is a maven application, we would use the below command to build the source code:

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
            echo 'If you are using a "manual" build mode for one or more of the' \
              'languages you are analyzing, replace this with the commands to build' \
              'your code, for example:'

            mvn clean install
    
        - name: Perform CodeQL Analysis
          uses: github/codeql-action/analyze@v3
          with:
            category: "/language:${{matrix.language}}"
```
</details>

## Lab 5 - Security Campaigns

#### Objective
Security campaigns are a way to group alerts and share them with developers, so you can collaborate to remediate vulnerabilities in the code. In this lab we will try creating a security campaign for identified security alerts in the default branch and see how the Campaigns feature in GHAS will help bring in focussed remediation effort from Dev teams. With this feature you can fix security alerts at scale by creating security campaigns and collaborating with developers to burn down your security backlog.

#### Steps

For this exercise, we will use the ORG level Security tab for creating `Campaigns`

 1. Navigate to the Security tab in your working Org
 2. Under the security tab, click on the `Campaigns` menu on the left hand pane
 3. In the subsequent menu, select an  existing template to create a Campaign
    1. Critial CodeQL Alerts
    2. Mitre top 10 KEV
    3. SQL Injection
    4. Cross-site Scripting
    5. or Create a custom filter for the campaign

<details>
  <summary> Animated Guide</summary>

![alt text](images/Security-Campaign-01.gif)

</details>

4. **Question :** Can you create a custom campaign for all the open vulnerabilities of severity `critical` and has `Autofix Suggestions` ?

 <details>
  <summary> Solution </summary>

![alt text](images/Security-Campaign-02.gif)

</details>  

## Lab 6 - Code Scanning for monorepos

#### Objective

In this lab, you will learn how to configure CodeQL for efficient monorepo scanning.

Monorepos are large in size, which often leads to longer scan times that can delay PR workflows. To optimize performance, teams often choose to split scans, focusing only on the relevant parts of the repository affected by code changes. The feasibility of splitting scans depends on the architecture of the monorepo.

This lab demonstrates the concept of monorepo scanning with CodeQL. However, the way it is ultimately divided will depend on factors such as the bottlenecks in the analysis, the tech stack, and the architecture.

We will be using the `mono-gallery` repository to do the exercises for this lab. 

Due to time and size constraints of real-world monorepos, mono-gallery is a smaller example, but its setup follows the same principles as a full-scale monorepo. This repository is a copy of the mona-gallery template and consists of multiple independent application tiers, each organized into separate directories:

- Directory: auth
    - Language: go
    - Build Mode: autbuild
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

These are the only directories within the `mono-gallery` that we're interested in scanning. Any changes outside these directories should be skipped. 

#### Steps

1. Navigate to your code scanning settings and switch from the default workflow to the advanced workflow. You'll be directed to save a workflow template in the `.github/workflows` directory. You will edit this workflow in the following steps.

    <details>
      <summary> Animated Guide</summary>
    
      ![alt text](images/default-to-advanced.gif)
    
    </details>


2. The workflow you are creating will contain three distinct jobs:
     - **detect_changes**: Identifies modified areas of your monorepo.
     - **analyze**: Performs the actual CodeQL analysis based on the detected changes.
     - **process_sarif**: Processes directories that have not changed. 
     
     Your task is to:
     1. Understand the YAML structure in the template below.
     2. Familiarize yourself with the GitHub Actions structure, which includes jobs, steps, and workflow triggers. You can read more about GitHub Actions [here](https://docs.github.com/en/actions)
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


3. Update the action template to detect changes on both `push` and `pull_request` events. This should be implemented in the `detect_changes` job within the `# TODO: Implement detection logic` section.

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

  4.  Update the action file to perform CodeQL analysis. Modify the job `analyze` to include CodeQL analysis for different components of the project using the category property.

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

   5. Modify the action file to handle the `process_sarif` job, ensuring that all required checks pass by submitting an empty SARIF report for unchanged components.

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
