# Module 3 - Code Scanning

---

## Lab 1 - Customising CodeQL Configuration

#### Objective
In this lab, you will learn how to customize your CodeQL workflow to enable the `security-and-quality` query suite.

#### Steps

1. Create the file `.github/codeql/codeql-config.yml` and enable the `security-and-quality` suite. This configuration file controls which queries CodeQL will run.

   <details>
     <summary>Hint</summary>

     A configuration file contains a `queries` key where you can specify additional queries as follows:

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

2. Enable your custom configuration in the CodeQL workflow file (for example, `.github/workflows/codeql.yml`).  
   - Reference the custom config file by using the `init` action’s `config-file` parameter.

   <details>
     <summary>Hint</summary>
      
     ```yaml
     - name: Initialize CodeQL
       uses: github/codeql-action/init@v3
       with:
         languages: ${{ matrix.language }}
         build-mode: ${{ matrix.build-mode }}
         config-file: ./.github/codeql/codeql-config.yml
     ```
   </details>

   <details>
     <summary>Solution</summary>

     ```yaml
     name: "CodeQL"
     on:
       push:
         branches: [ "main" ]
       pull_request:
         branches: [ "main" ]
     jobs:
       analyze:
         runs-on: ubuntu-latest
         steps:
           - name: Checkout repository
             uses: actions/checkout@v2

           - name: Initialize CodeQL
             uses: github/codeql-action/init@v2
             with:
               languages: js,go
               config-file: ./.github/codeql/codeql-config.yml

           - name: Perform CodeQL Analysis
             uses: github/codeql-action/analyze@v2
     ```
   </details>

#### Discussion Points
- Review the alerts, are you getting more results?
- It was also possible to not use a custom config file and just add `queries:security-and-quality` in the CodeQL workflow file, like so:

```yaml
   # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: ${{ matrix.language }}
        build-mode: ${{ matrix.build-mode }}
        queries: security-and-quality
```
In what cases would we just use the inline method above versus having a config file?

---

## Lab 2 - Running Custom Queries

#### Objective
In this lab, you will learn how to run your own custom CodeQL queries for different languages. You will also learn how to include these queries in your CodeQL configuration and run as a part of your workflow.

#### Steps

1. Create a QL pack file and query for Go  
   - Add `custom-queries/go/qlpack.yml`:
     ```yaml
     ---
     library: false
     warnOnImplicitThis: false
     name: queries
     version: 0.0.1
     dependencies:
       codeql/go-all: "*"
     ```
     This file creates a [QL query pack](https://help.semmle.com/codeql/codeql-cli/reference/qlpack-overview.html) used to organize query files and their dependencies.

   - Create the query file, for example, `custom-queries/go/jwt.ql`:

     ```ql
     /**
      * @name Missing token verification
      * @description Missing token verification
      * @id go/jwt-sign-check
      * @kind problem
      * @problem.severity warning
      * @precision high
      * @tags security
      */
     import go

     from FuncLit f
     where
       f.getParameter(0).getType() instanceof PointerType and
       f.getParameter(0).getType().(PointerType).getBaseType().getName() = "Token" and
       f.getParameter(0).getType().(PointerType).getBaseType().getPackage().getName() = "jwt" and
       not exists(TypeExpr t |
         f.getBody().getAChild*() = t and
         t.getType().getName() = "SigningMethodHMAC" and
         t.getType().getPackage().getName() = "jwt"
       )
     select f, "This function should be using jwt.SigningMethodHMAC"
     ```

2. Create another QL pack file for JavaScript 
   - For example, `custom-queries/js/qlpack.yml`:

     ```yaml
     ---
     library: false
     warnOnImplicitThis: false
     name: queries
     version: 0.0.1
     dependencies:
       codeql/javascript-all: "*"
     ```

- Add a custom Vue-based XSS detection query
   - Create a suitable `.ql` file, for example: `custom-queries/js/vue-xss.ql`.

     ```ql
     /**
      * @name Client-side cross-site scripting
      * @description Writing user input directly to the DOM allows for
      *              a cross-site scripting vulnerability.
      * @kind path-problem
      * @problem.severity error
      * @security-severity 6.1
      * @precision high
      * @id js/vue-xss
      * @tags security
      *       external/cwe/cwe-079
      *       external/cwe/cwe-116
      */

     import javascript
     import semmle.javascript.security.dataflow.DomBasedXssQuery
     import semmle.javascript.security.dataflow.DomBasedXssCustomizations
     import DataFlow::PathGraph

     // Unprecise Axios Source
     class AxiosSource extends DomBasedXss::Source {
       AxiosSource() {
         this.(DataFlow::PropRead).getPropertyName() = "data" and
         exists(DataFlow::ParameterNode response |
           response.getName() = "response" |
           response.flowsTo(this.(DataFlow::PropRead).getBase())
         )
       }
     }

     // ...rest of the query omitted for brevity...
     ```

4. Create a custom [query suite](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/) in a file named `mycustom-suite.qls` inside the `custom-queries` directory. The custom query suite should run the custom `vue-xss.ql` and `jwt.ql` queries (created previously) and also include the standard `security-and-quality` suites for JavaScript, Java, Python, and Go.
By default, the JavaScript `security-and-quality` suite includes a built-in XSS query (`javascript/xss`). Because the `vue-xss.ql` query covers the same ground, the default query should be explicitly excluded the new suite.

<details>

<summary>Hint</summary>

You can import existing query suites by specifying:

```yaml
- import: codeql-suites/{language}-security-and-quality.qls
  from: codeql-{language}
```

This uses the `import` keyword to include another suite, and the `from` keyword to reference the QL pack containing that suite (for example, `codeql-javascript`). In your custom suite, you can also add local queries by specifying:
```yaml
- queries: path-to-local-queries
```
To exclude certain queries,use:
```yaml
- exclude:
    id:
      - path/to/query
```

</details>


<details>

 <summary>Solution</summary>

```yaml
       - queries: custom-queries
       # Reusing existing QL Pack
       - import: codeql-suites/javascript-security-and-quality.qls
         from: codeql-javascript
       - import: codeql-suites/java-security-and-quality.qls
         from: codeql-java
       - import: codeql-suites/python-security-and-quality.qls
         from: codeql-python
       - import: codeql-suites/go-security-and-quality.qls
         from: codeql-go
```




</details>




4. Update the CodeQL configuration file `.github/codeql/codeql-config.yml` to include your custom queries. 
   - Op.

   <details>
     <summary>Need Help? Here's a hint</summary>

     The `uses` key supports paths relative to your repository. You can also disable the default queries using `disable-default-queries: true` if you want full control.
   </details>

   <details>
     <summary>Solution</summary>

     ```yaml
     name: "My CodeQL config"

     disable-default-queries: true

     queries:
       - uses: security-and-quality
       - uses: ./custom-queries/code-scanning.qls
       - uses: ./custom-queries/go/jwt.ql

     paths-ignore:
       - '**/test/**'
     ```
   </details>

   <details>
     <summary>Alternate Example</summary>

     <details>
       <summary>Solution</summary>

       ```yaml
       - queries: 
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



#### Discussion Points
- When would you create a custom query pack vs. rely on the default security suites?
- Which team members should be responsible for creating or updating custom queries?
- How could you test new custom queries before enabling them in production pipelines?
