# Module 3 - Code Scanning 


### Lab X - Customising CodeQL Configuration 

By default, CodeQL uses a selection of queries that provide high quality security results. However, you might want to change this behavior to:

- Include broader set of security queries.
- Include queries with a lower signal to noise ratio to detect more potential issues.
- To exclude queries in the default pack because they generate false positives for your architecture.
- Include custom queries written for your project.

#### Objective
In this lab, you will learn how to customise your workflow to use the `security-and-quality` suite, as well as include and exclude queries for your configuration. 
With CodeQL code scanning, you can select a specific group of CodeQL queries, called a CodeQL query suite, to run against your code. 

#### Steps 
1. Create the file `.github/codeql/codeql-config.yml` and enable the `security-and-quality` suite.
   <details>
     <summary>Need Help? Here's a hint</summary>
     A configuration file contains a key `queries` where you can specify additional queries as follows
     
      ```yaml   
                name: "My CodeQL config"
                queries:
                  - uses: <insert your query suite>
      ```
   </details>

   <details>
     <summary>Solution</summary>
     
     ```yaml
                name: "My CodeQL config"
                queries:
                  - uses: security-and-quality
      ```
   </details>

2. Enable your custom configuration in the code scanning workflow file `.github/codeql/codeql-config.yml`

<details>

<summary>Hint</summary>

The `init` action supports a `config-file` parameter to specify a configuration file.

</details>

<details>

<summary>Solution</summary>



  
</details>


   2. Our application has a XSS vulnerability that is not being identified by CodeQL by default. Add a custom query to include in our analysis and exclude the default one.
  CodeQL query suites provide a way of selecting queries, based on their filename, location on disk or in a CodeQL pack, or metadata properties. Create query suites for the queries that you want to frequently use in your CodeQL analyses.

Query suites allow you to pass multiple queries to CodeQL without having to specify the path to each query file individually. Query suite definitions are stored in YAML files with the extension .qls. A suite definition is a sequence of instructions,
where each instruction is a YAML mapping with (usually) a single key. The instructions are executed in the order they appear in the query suite definition. After all the instructions in the suite definition have been executed, the result is a set of selected queries.

Add t
  
  

   <details>
     <summary></summary>
     <details>
<summary>Solution</summary>

```yaml
# Reusing existing QL Pack
- import: codeql-suites/javascript-security-and-quality.qls
  from: codeql-javascript
- import: codeql-suites/java-security-and-quality.qls
  from: codeql-java
- import: codeql-suites/python-security-and-quality.qls
  from: codeql-python
- import: codeql-suites/go-security-and-quality.qls
  from: codeql-go
- exclude:
  id:
    - javascript/xss
```
</details>
   </details>


#### Steps
1. [Actionable instruction.]
   - [Additional clarification if necessary.]
2. [Next step.]
   <details>
     <summary>Need Help? View Screenshot</summary>
     ![alt text](path/to/image.png)
   </details>

#### Discussion Points
- [Question 1]
- [Question 2]
- [Question 3]
