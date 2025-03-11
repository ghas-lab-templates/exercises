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
     <summary>Hint</summary>

   </details>


2. The workflow you are creating will contain two distinct jobs:
     - **detect_changes**: Identifies modified areas of your monorepo.
     - **analyze**: Performs the actual CodeQL analysis based on the detected changes.
     
     Your task is to:
     1. Create the YAML structure outlined above.
     2. Familiarize yourself with the GitHub Actions structure, which includes jobs, steps, and workflow triggers.

You can read more about GitHub Actions [here](https://docs.github.com/en/actions)

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
                  .github/scripts/empty.sarif
                sparse-checkout-cone-mode: false
            # TODO: Implement CodeQL analysis steps

    ```    
   </details>


   3. Navigate to `.github/scripts` and review the scripts:
      - Understand the role of `process.awk` in mapping file changes to directories and languages.
      - Verify how JSON outputs for changes (`matrix`) and unchanged directories (`matrix_no_changes`) are generated.

   <details>
     <summary>Hint</summary>

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

   - Add a custom Vue-based XSS `.ql` file, for example: `custom-queries/js/vue-xss.ql`.

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
          exists(DataFlow::ParameterNode response | response.getName() = "response" |
            response.flowsTo(this.(DataFlow::PropRead).getBase())
          )
        }
      }
      
      class VForAttribute extends DataFlow::Node {
        HTML::Attribute attr;
      
        VForAttribute() {
          this.(DataFlow::HtmlAttributeNode).getAttribute() = attr and attr.getName() = "v-for"
        }
      
        /**
         * Gets the HTML attribute of this sink.
         */
        HTML::Attribute getAttr() { result = attr }
      
        string getForAlias() { result = attr.getValue().regexpCapture("(.*)\\s+in\\s+(.*)", 1).trim() }
      
        string getForArraySource() {
          result = attr.getValue().regexpCapture("(.*)\\s+in\\s+(.*)", 2).trim()
        }
      
        Vue::Component getComponent() {
          result.getTemplateElement().(Vue::Template::HtmlElement).getElement() = this.getAttr().getRoot()
        }
      
        DataFlow::Node getAForArraySourceValue() {
          exists(DataFlow::PropWrite propWrite |
            resolveRefForArraySourceValue(this, this.getForArraySource(), propWrite) and
            result = propWrite.getRhs()
          )
        }
      }
      
      predicate resolveRefForArraySourceValue(
        VForAttribute vfor, string accessPath, DataFlow::PropRef propRef
      ) {
        // accesssPath must be a prefix of the for array source.
        exists(int dotOffsets, string arraySrc |
          arraySrc = vfor.getForArraySource() and dotOffsets = arraySrc.indexOf(".")
        |
          (
            arraySrc.prefix(dotOffsets) = accessPath
            or
            arraySrc = accessPath
          ) and
          accessPath != "this"
        ) and
        (
          // Base case: foo or this.foo
          exists(string prop | prop = accessPath.replaceAll("?", "").replaceAll("this.", "") |
            propRef = vfor.getComponent().getAnInstanceRef().getAPropertyReference(prop)
          )
          or
          // Induction step: foo.bar or this.foo.bar
          exists(DataFlow::PropRef basePropRef, string baseAccessPath, string prop |
            baseAccessPath = accessPath.prefix(max(int idx | idx = accessPath.indexOf("."))) and
            prop = accessPath.suffix(max(int idx | idx = accessPath.indexOf(".")) + 1).replaceAll("?", "") and
            resolveRefForArraySourceValue(vfor, baseAccessPath, basePropRef) and
            propRef.getPropertyName() = prop and
            (
              propRef.getBase() = basePropRef
              or
              // A more precise source can be identified when the source of this.gallery, reponse.data, is flowing through an intermediate variable
              // to the base of a property reference where the property name matches our current property.
              // e.g.,
              // const gallery = response.data
              // axios.get("http://localhost:8081/gallery/art").then((response) => {
              //    gallery.art = response.data
              //    this.gallery = gallery
              basePropRef.(DataFlow::PropWrite).getRhs().getALocalSource().flowsTo(propRef.getBase())
            )
          )
        )
      }
      
      class VHtmlAttributeStep extends TaintTracking::SharedTaintStep {
        override predicate viewComponentStep(DataFlow::Node pred, DataFlow::Node succ) {
          succ.(VForAttribute).getAForArraySourceValue() = pred
          or
          exists(VForAttribute vfor, Vue::VHtmlAttribute vhtml | pred = vfor and succ = vhtml |
            vfor.getAttr().getElement() = vhtml.getAttr().getElement().getParent+() and
            vhtml.getAttr().getValue().prefix(vhtml.getAttr().getValue().indexOf(".", 0, 0)) =
              vfor.getForAlias()
          )
        }
      }
      
      from DataFlow::Configuration cfg, DataFlow::PathNode source, DataFlow::PathNode sink
      where cfg.hasFlowPath(source, sink)
      select sink.getNode(), source, sink,
        sink.getNode().(Sink).getVulnerabilityKind() + " vulnerability due to $@.", source.getNode(),
        "user-provided value"
      
        ```

3. Create a custom [query suite](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/) in a file named `custom-suite.qls` inside the `custom-queries` directory. The custom query suite should run the custom `vue-xss.ql` and `jwt.ql` queries (created previously) and also include the standard `security-and-quality` suites for JavaScript, Java, Python, and Go.
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




4. Update the CodeQL configuration file `.github/codeql/codeql-config.yml` to include use the custom query suite. 

   <details>
     <summary>Need Help? Here's a hint</summary>

     The `uses` key supports paths relative to your repository. You can also disable the default queries using `disable-default-queries: true` so that there isn't a duplication in alerts. 
   </details>

   <details>
      
     <summary>Solution</summary>

     ```yaml
     name: "My CodeQL config"

     disable-default-queries: true

     queries:
       - uses: ./custom-queries/custom-suite.qls
     ```
     
   </details>


#### Discussion Points
- As your application grows, how do you foresee maintaining or extending these custom queries? Who should own that process?
- What criteria would you use to decide whether a new security concern warrants writing an entirely new custom query versus customizing an existing one?
