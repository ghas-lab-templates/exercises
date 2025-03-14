# Module 3 - Code Scanning

---

## Lab 1 - Code Scanning for monorepos

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

