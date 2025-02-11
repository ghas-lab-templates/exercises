# Module 3 - Code Scanning

### Lab X - Customising CodeQL Configuration 

By default, CodeQL uses a selection of queries that provide high quality security results. However, you might want to change this behavior to:

- Include a broader set of security queries.
- Include queries with a lower signal-to-noise ratio to detect more potential issues.
- Exclude queries in the default pack that generate false positives for your architecture.
- Include custom queries written for your project.

#### Objective
In this lab, you will learn how to customize your workflow to use the `security-and-quality` suite, as well as include and exclude queries for your configuration. With CodeQL code scanning, you can select a specific group of CodeQL queries, called a CodeQL query suite, to run against your code.

#### Steps
1. Create the file `.github/codeql/codeql-config.yml` and enable the `security-and-quality` suite.
   - This configuration file controls which queries CodeQL will run.
   <details>
     <summary>Need Help? Here's a hint</summary>

     A configuration file contains a key `queries` where you can specify additional queries as follows:

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

2. Enable your custom configuration in the code scanning workflow file (e.g., `.github/workflows/codeql.yml`).
   - Reference the custom config file by using the `init` action’s `config-file` parameter.
   <details>
     <summary>Hint</summary>

     ```yaml
     - name: Initialize CodeQL
       uses: github/codeql-action/init@v2
       with:
         languages: <language(s)>
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

3. Add a custom query to detect a potential XSS vulnerability and exclude a default query.
   - CodeQL query suites let you include or exclude queries based on filename, location, or metadata properties.
   - You can create query suites for queries you frequently use in CodeQL analyses.

   - Create a QL pack file (e.g., `custom-queries/go/qlpack.yml`):
     ```yaml
     name: my-go-queries
     version: 0.0.0
     libraryPathDependencies:
       - codeql-go
     ```
     This file creates a [QL query pack](https://help.semmle.com/codeql/codeql-cli/reference/qlpack-overview.html) used to organize query files and their dependencies.
   
   - **Create the actual query file (e.g., `custom-queries/go/jwt.ql`):**
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

4. **Update the CodeQL configuration file `.github/codeql/codeql-config.yml` to include your custom query and exclude the default one.**
   <details>
     <summary>Need Help? Here's a hint</summary>

     The `uses` key supports paths relative to your repository. You can also disable the default queries using `disable-default-queries: true` if you want full control over which queries are used.
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

5. **Add another custom query for Vue-based XSS detection.**  
   You can place this query in a suitable `.ql` file (for example, `custom-queries/js/vue-xss.ql`).  

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

   class VForAttribute extends DataFlow::Node {
     HTML::Attribute attr;

     VForAttribute() {
       this.(DataFlow::HtmlAttributeNode).getAttribute() = attr and
       attr.getName() = "v-for"
     }

     HTML::Attribute getAttr() { result = attr }

     string getForAlias() {
       result = attr.getValue().regexpCapture("(.*)\\s+in\\s+(.*)", 1).trim()
     }

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
     exists(int dotOffsets, string arraySrc |
       arraySrc = vfor.getForArraySource() and dotOffsets = arraySrc.indexOf(".")
       |
       (
         arraySrc.prefix(dotOffsets) = accessPath
         or
         arraySrc = accessPath
       )
       and
       accessPath != "this"
     )
     and
     (
       // Base case: foo or this.foo
       exists(string prop |
         prop = accessPath.replaceAll("?", "").replaceAll("this.", "") |
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
           basePropRef.(DataFlow::PropWrite).getRhs().getALocalSource().flowsTo(propRef.getBase())
         )
       )
     )
   }

   class VHtmlAttributeStep extends TaintTracking::SharedTaintStep {
     override predicate viewComponentStep(DataFlow::Node pred, DataFlow::Node succ) {
       succ.(VForAttribute).getAForArraySourceValue() = pred
       or
       exists(VForAttribute vfor, Vue::VHtmlAttribute vhtml |
         pred = vfor and succ = vhtml |
         vfor.getAttr().getElement() = vhtml.getAttr().getElement().getParent+() and
         vhtml.getAttr().getValue().prefix(
           vhtml.getAttr().getValue().indexOf(".", 0, 0)
         ) = vfor.getForAlias()
       )
     }
   }

   from DataFlow::Configuration cfg, DataFlow::PathNode source, DataFlow::PathNode sink
   where cfg.hasFlowPath(source, sink)
   select sink.getNode(), source, sink,
     sink.getNode().(Sink).getVulnerabilityKind() + " vulnerability due to $@.", source.getNode(),
     "user-provided value"
