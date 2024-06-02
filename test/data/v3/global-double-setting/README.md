## How to run locally?
* `helm unittest .`

## Notes
* test suite's vales
  * if you add several, the ⚠️last element takes precedence⚠️ -- 'override_test.yaml' & 'configmap_test_another.yaml' --
    * Problems:
      * Problem1: 'configmap_test_another.yaml' not executed
        * Attempt1: Rename testSuite's it
        * Attempt2: Use 1 values' entry
        * Attempt3: Add testSuite's suite
        * Solution: Rename the fileName
        * Reason: Check 'defaultFilePattern' in the code