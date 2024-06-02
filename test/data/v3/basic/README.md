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
* "tests/ingress_test.yaml"
  * YAML / JSONPath support