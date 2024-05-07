# How to run tests?
* `helm unittest .`

## Notes
* "/tests/configmap_test.yaml"
  * asserts used are
    * `contains`
    * `notMatchRegex`
* "/tests/deployment_test.yaml"
  * asserts used are
    * `isSubset`