# How to run tests?
* `helm unittest .`
* `helm unittest -d true .`
  * check in the logs, the rendering process of the manifests

## Notes
* "/tests/configmap_test.yaml"
  * asserts used are
    * `contains`
    * `notMatchRegex`
* "/tests/deployment_test.yaml"
  * asserts used are
    * `isSubset`
    * `lengthEqual`
  * `testSuite.set`
    * passed
    * takes precedence vs `testSuite.values`
  * `testSuite.values`
    * if you add several ons -> last ones take precedence
  * `documentIndex`
* "tests/ingress_test.yaml"
  * YAML / JSONPath support
* test suite completion and validation
  * via
    * [inlined schema](https://github.com/redhat-developer/yaml-language-server?tab=readme-ov-file#using-inlined-schema)
      * Problems:
        * Problem1: NOT work
          * Attempt1: `# yaml-language-server: $schema=https://github.com/helm-unittest/helm-unittest/blob/main/schema/helm-testsuite.json`
          * Attempt2: `# yaml-language-server: $schema=https://raw.githubusercontent.com/helm-unittest/helm-unittest/main/schema/helm-testsuite.json`
          * Attempt3: Schemas for Intellij
          * Solution: TODO: