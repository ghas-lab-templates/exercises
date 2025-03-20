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

11. As the final step once you have validated all the config iptions in the `Default scan`, you can go ahead and click on the `Enable COdeQL` button
12. This will start the CodeQL analysis using the GitHub Hosted Runners. You can validate the scan progress in the `Actions` tab
13. Once the scan is completed for all the listed technologies, you can navigate to the `Security` tab and then under `Code scanning` you can see all the CodeQL vulnerabilities

<details>
  <summary> Animated Guide</summary>

![alt text](images/code-scan-default-settings-3.gif)

</details>


## Lab 2 - Advanced Configuration with custom build command

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

## Lab 3 - Security Campaigns

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
