# Testing Document

This document describes how to define your tests with YAML

- [Test Suite](#test-suite)
- [Test Job](#test-job)
- [Assertion](#assertion)
  - [Assertion Types](#assertion-types)
  - [Antonym and `not`](#antonym-and-not)

## Test Suite

* Collection of tests defined in 1! file / same
  * purpose and
  * scope 
* Structure

```yaml
suite: test deploy and service
values:
  - overallValues.yaml
set:
  image.pullPolicy: Always
templates:
  - templates/deployment.yaml
  - templates/web/service.yaml
release:
  name: my-release
  namespace: my-namespace
  revision: 1
  upgrade: true
capabilities:
  majorVersion: 1
  minorVersion: 10
  apiVersions:
    - br.dev.local/v2
chart:
  version: 1.0.0
  appVersion: 1.0.0
tests:
  - it: should test something
    ...
```

- **suite**:
  - *string*
    -  optional
  - suite name / shown on test result output

- **values**:
  - *string[]*
    - optional
  - Relative file path -- to the -- test suite values
    - == (in helm commands) `helm ... -f --valueFile` 
      - -> if you add several, the ⚠️last element takes precedence⚠️

- **set**: 
  - *object of any*
    - optional
  - -- alternative to pass by -- `values`
    - ⚠️takes precedence vs passed by `values` ⚠️
  - == (in helm command) `helm install --set key=value`

- **templates**: 
  - *string[]*
    - recommended
    - relative path to the templates to test
      - if templates are in sub-folders -> use '/' 
      - `templates/` can be omitted
      - `*` allows testing multiple templates / without listing them one-by-one
  - Template files to test in this suite
    - although full chart will be rendered 
    - Partial templates ( == prefixed with and `_` or have the .tpl extension) are added automatically
      - even if it is in a templates sub-folder -- TODO: How to check ? --

- **release**:
  - *object*  
    - Define the `{{ .Release }}` object
    - optional
    - **name**
      - *string* 
      - optional
        - `"RELEASE-NAME"` default one
    - **namespace**
      - *string*
      - optional
        - `"NAMESPACE"` default
      - Namespace which release be installed to
    - **revision**:
      - *int*
      - optional
        - `0` default one
      - Revision of current build 
    - **upgrade**:
      - *bool*
      - optional
        - `false` default one

- **capabilities**:
  - *object*
    - Define the `{{ .Capabilities }}` object
    - optional
      - **majorVersion**:
        - *int*
        - optional 
          - major version / set by helm is the default one 
        - Kubernetes major version, 
      - **minorVersion**:
        - *int*
        - optional
          - minor version / set by helm is the default one 
        - Kubernetes minor version
      - **apiVersions**:
        - *string[]*
        - optional
          - versionset / defined by kubernetes version is the default one 
        - Set of Kubernetes versions

- **chart**:
  - *object*
    - Define the `{{ .Chart }}` object
    - optional
    - **version**:
      - *string*
      - optional
        - Version / set in the Chart is the default one 
      - Semantic version of the chart
    - **appVersion**:
      - *string*
      - optional
        - App-version / set in the Chart is the default one
      - App-version of the chart

- **tests**:
  - *testJob[]*
  - required
  - Check [Test Job](#test-job)

## Test Job

* == base unit of testing
  * allows
    * if you run a test job -> 
      * chart is **rendered** (-- `helm unittest -d true ...` --) and
      * validated -- against -- assertions defined in the test 

```yaml
...
tests:
  - it: should pass
    values:
      - ./values/staging.yaml
    set:
      image.pullPolicy: Always
      resources:
        limits:
          memory: 128Mi
    template: deployment.yaml
    documentIndex: 0
    documentSelector: 
      path: metadata.name
      value: my-service-name    
    release:
      name: my-release
      namespace:
      revision: 9
      upgrade: true
    capabilities:
      majorVersion: 1
      minorVersion: 12
      apiVersions:
        - custom.api/v1
    chart:
      version: 1.0.0
      appVersion: 1.0.0
    asserts:
      - equal:
          path: metadata.name
          value: my-deploy
```

- **it**:
  - *string*
    - recommended
  - Test name

- **values**:
  - *[]string*
    - optional
  - == test suite's `values`
    - override / defined in test suite!!

- **set**:
  - *object of any*
    - optional
  - == test suite's `set`
    - override / defined in test suite!! 

- **template**:
  - *string*
    - optional
  - == templates BUT 1!
- **templates**:
  - *[]string*
    - optional
  - == test suite's `templates`
    - override / defined in test suite!!

- **documentIndex**:
  - *int*
    - optional
    - -1
      - by default
      - == ALL kind resources specified in template will be tested
  - == index of rendered documents (divided by `---` / file) to be tested 

- **documentSelector**:
  - *DocumentSelector*
    - -- alternative to -- `documentIndex`
    - optional  
      - **path**:
        - *string*.
        - path to assert
        - Yaml / JsonPath Support
          - allows
            - filtering on multiple fields
      - **value**:
        - *any*
          - == complete yaml objects
        - The expected value.

- **release**:
  - == test suite's `release`

- **capabilities**: 
  - == test suite's `capabilities`

- **chart**: 
  - == test suite's `chart`

- **asserts**:
  - *assertion[]*
    - required
    - Check [Assertion](#assertion)

---

## Assertion

* TestJob's assertions
* allows
  * providing values -> validate the manifests 

```yaml
...
tests:
  - it: ...
    asserts:
      - assertionType:
          assertionTypeArguments
        not: 
        template: ...
        documentIndex: ..
        documentSelector: ...
      - ...
```

- **assertionType**
  - Check [Assertion Types](#assertion-types)
  - 1! / `asserts` 

- **not**: 
  - *bool*
    - optional
    - `false` default
    - if `true` ->  assert contrarily

- **template**:
  - *string*
    - optional
  - == testJob's `template` 
    - override the testJob's `template` & testSuite's `template`

- **documentIndex**:
  - *int*
    - optional
  - == testJob's `documentIndex` 
    - override testJob's one

- **documentSelector**:
  - *DocumentSelector*
    - optional
  - == testJob's `documentSelector`
    - override testJob's one


---

### Assertion Types

Available assertion types are listed below:

| Assertion Type | Parameters                                                                                                                                                                                                                                                                                                                       | Description                                                                                                                                                                                                          | Example |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| `containsDocument` | **kind**: *string*. Expected `kind` of manifest.<br/> **apiVersion**: *string*. Expected `apiVersion` of manifest.<br/>**name**: *string, optional*. The value of the `metadata.name`.<br/>**namespace**: *string, optional*. The value of the `metadata.namespace`.<br/>**any**: *bool, optional*. ignores any other documents. | Asserts the documents rendered by the `kind` and `apiVersion` specified.                                                                                                                                             | <pre>containsDocument:<br/>  kind: Deployment<br/>  apiVersion: apps/v1<br/>  name: foo<br/>  namespace: bar</pre> |
| `contains` | **path**: *string*. The `set` path to assert, the value must be an *array*. <br/>**content**: *any*. The content to be contained.<br/>**count**: *int, optional*. The count of content to be contained.<br/>**any**: *bool, optional*. ignores any other values within the found content.                                        | Assert the array specified in **path**, contains the **content**.                                                                                                                                                    |<pre>contains:<br/>  path: spec.ports<br/>  content:<br/>    name: web<br/>    port: 80<br/>    targetPort: 80<br/>    protocol:TCP<br/><br/>contains:<br/>  path: spec.ports<br/>  content:<br/>    name: web<br/>  count: 1<br/>  any: true<br/></pre> |
| `notContains` | **path**: *string*. The `set` path to assert, the value must be an *array*. <br/>**content**: *any*. The content NOT to be contained.<br/>**any**: *bool, optional*. ignores any other values within the found content.                                                                                                          | Assert the array specified in **path**, NOT contains the **content**.                                                                                                                                                |<pre>notContains:<br/>  path: spec.ports<br/>  content:<br/>    name: server<br/>    port: 80<br/>    targetPort: 80<br/>    protocol: TCP<br/><br/>notContains:<br/>  path: spec.ports<br/>  content:<br/>    name: web<br/>  any: true<br/></pre> |
| `equal` | **path**: *string*. The `set` path to assert.<br/>**value**: *any*. The expected value.<br/>**decodeBase64**: *bool, optional*. Decode the base64 before checking                                                                                                                                                                | Assert the value specified in **path**, equal to the **value**.                                                                                                                                                      | <pre>equal:<br/>  path: metadata.name<br/>  value: my-deploy</pre> |
| `notEqual` | **path**: *string*. The `set` path to assert.<br/>**value**: *any*. The value expected not to be. <br/>**decodeBase64**: *bool, optional*. Decode the base64 before checking                                                                                                                                                     | Assert the value specified in **path**, NOT equal to the **value**.                                                                                                                                                  | <pre>notEqual:<br/>  path: metadata.name<br/>  value: my-deploy</pre> |
| `equalRaw` | <br/>**value**: *string*. Assert the expected value in a NOTES.txt file.                                                                                                                                                                                                                                                         | Assert equal to the **value**.                                                                                                                                                                                       | <pre>equalRaw:<br/>  value: my-deploy</pre> |
| `notEqualRaw` | <br/>**value**: *string*. Assert the expected value in a NOTES.txt file not to be.                                                                                                                                                                                                                                               | Assert equal NOT to the **value**.                                                                                                                                                                                   | <pre>notEqualRaw:<br/>  value: my-deploy</pre> |
| `exists`<br/>(deprecates `isNotNull`) | **path**: *string*. The `set` path to assert.                                                                                                                                                                                                                                                                                    | Assert if the specified **path** `exists`.                                                                                                                                                                           |<pre>exists:<br/>  path: spec.strategy</pre> |
| `notExists`<br/>(deprecates `isNull`) | **path**: *string*. The `set` path to assert.                                                                                                                                                                                                                                                                                    | Assert if the specified **path** NOT `exists`.                                                                                                                                                                       |<pre>notExists:<br/>  path: spec.strategy</pre> |
| `failedTemplate` | **errorMessage**: *string*. The (human readable) `errorMessage` that should occur.                                                                                                                                                                                                                                               | Assert the value of **errorMessage** is the same as the human readable template rendering error. Also allows to match an error that would happen before template execution (ex: validation of values against schema) | <pre>failedTemplate:<br/>  errorMessage: Required value<br/></pre> |
| `notFailedTemplate` |                                                                                                                                                                                                                                                                                                                                  | Assert that no failure occurs while templating.                                                                                                                                                                      | <pre>notFailedTemplate: {}<br/></pre> |
| `hasDocuments` | **count**: *int*. Expected count of documents rendered.                                                                                                                                                                                                                                                                          | Assert the documents count rendered by the `template` specified. The `documentIndex` or `documentSelector` option is ignored here.                                                                                   | <pre>hasDocuments:<br/>  count: 2</pre> |
| `isAPIVersion` | **of**: *string*. Expected `apiVersion` of manifest.                                                                                                                                                                                                                                                                             | Assert the `apiVersion` value **of** manifest, is equilevant to:<br/><pre>equal:<br/>  path: apiVersion<br/>  value: ...<br/>                                                                                        | <pre>isAPIVersion:<br/>  of: v2</pre> |
| `isKind` | **of**: *String*. Expected `kind` of manifest.                                                                                                                                                                                                                                                                                   | Assert the `kind` value **of** manifest, is equilevant to:<br/><pre>equal:<br/>  path: kind<br/>  value: ...<br/>                                                                                                    | <pre>isKind:<br/>  of: Deployment</pre> |
| `isNullOrEmpty`<br/>*`isEmpty`* | **path**: *string*. The `set` path to assert.                                                                                                                                                                                                                                                                                    | Assert the value of specified **path** is null or empty (`null`, `""`, `0`, `[]`, `{}`).                                                                                                                             |<pre>isNullOrEmpty:<br/>  path: spec.tls</pre> |
| `isNotNullOrEmpty`<br/>*`isNotEmpty`* | **path**: *string*. The `set` path to assert.                                                                                                                                                                                                                                                                                    | Assert the value of specified **path** is NOT null or empty (`null`, `""`, `0`, `[]`, `{}`).                                                                                                                         |<pre>isNotNullOrEmpty:<br/>  path: spec.selector</pre> |
| `isSubset` | **path**: *string*. The `set` path to assert, the value must be an *object*. <br/>**content**: *any*. The content to be contained.                                                                                                                                                                                               | Assert the object as the value of specified **path** that contains (!= equal) the **content**.                                                                                                                       |<pre>isSubset:<br/>  path: spec.template<br/>  content:<br/>    metadata: <br/>    labels: <br/>        app: basic<br/>        release: MY-RELEASE<br/></pre> |
| `isNotSubset` | **path**: *string*. The `set` path to assert, the value must be an *object*. <br/>**content**: *any*. The content NOT to be contained.                                                                                                                                                                                           | Assert the object as the value of specified **path** that NOT contains the **content**.                                                                                                                              |<pre>isSubset:<br/>  path: spec.template<br/>  content:<br/>    metadata: <br/>    labels: <br/>        app: basic<br/>        release: MY-RELEASE<br/></pre> |
| `lengthEqual` | **path**: *string, optional*. The `set` path to assert the count of array values. <br/>**paths**: *string, optional*. The `set` array of paths to assert the count validation of the founded arrays. <br/>**count**: *int, optional*. The count of the values in the array.                                                      | Assert the **count** of the **path** or **paths** to be equal.                                                                                                                                                       |<pre>lengthEqual:<br/>  path: spec.tls<br/>  count: 1<br/></pre> |
| `notLengthEqual` | **path**: *string, optional*. The `set` path to assert the count of array values. <br/>**paths**: *string, optional*. The `set` array of paths to assert the count validation of the founded arrays. <br/>**count**: *int, optional*. The count of the values in the array.                                                      | Assert the **count** of the **path** or **paths** NOT to be equal.                                                                                                                                                   |<pre>notLengthEqual:<br/>  path: spec.tls<br/>  count: 1<br/></pre> |
| `matchRegex` | **path**: *string*. The `set` path to assert, the value must be a *string*. <br/>**pattern**: *string*. The regex pattern to match (without quoting `/`).<br/>**decodeBase64**: *bool, optional*. Decode the base64 before checking                                                                                              | Assert the value of specified **path** match **pattern**.                                                                                                                                                            | <pre>matchRegex:<br/>  path: metadata.name<br/>  pattern: -my-chart$</pre> |
| `notMatchRegex` | **path**: *string*. The `set` path to assert, the value must be a *string*. <br/>**pattern**: *string*. The regex pattern NOT to match (without quoting `/`).<br/> **decodeBase64**: *bool, optional*. Decode the base64 before checking                                                                                                                                                                   | Assert the value of specified **path** NOT match **pattern**.<br/>                                                                              | <pre>notMatchRegex:<br/>  path: metadata.name<br/>  pattern: -my-chat$</pre> |
| `matchRegexRaw` | **pattern**: *string*. The regex pattern to match (without quoting `/`) in a NOTES.txt file.                                                                                                                                                                                                                                     | Assert the value match **pattern**.                                                                                                                                                                                  | <pre>matchRegexRaw:<br/>  pattern: -my-notes$</pre> |
| `notMatchRegexRaw` | **pattern**: *string*. The regex pattern NOT to match (without quoting `/`) in a NOTES.txt file.                                                                                                                                                                                                                                 | Assert the value NOT match **pattern**.                                                                                                                                                                              | <pre>notMatchRegexRaw:<br/>  pattern: -my-notes$</pre> |
| `matchSnapshot` | **path**: *string*. The `set` path for snapshot.                                                                                                                                                                                                                                                                                 | Assert the value of **path** is the same as snapshotted last time. Check [doc](./README.md#snapshot-testing) below.                                                                                                  | <pre>matchSnapshot:<br/>  path: spec</pre> |
| `matchSnapshotRaw` |                                                                                                                                                                                                                                                                                                                                  | Assert the value in the NOTES.txt is the same as snapshotted last time. Check [doc](./README.md#snapshot-testing) below.                                                                                             | <pre>matchSnapshotRaw: {}<br/></pre> |

### Antonym and `not`

Notice that there are some antonym assertions, the following two assertions actually have same effect:
```yaml
- equal:
    path: kind
    value: Pod
  not: true
# works the same as
- notEqual:
    path: kind
    value: Pod
```
