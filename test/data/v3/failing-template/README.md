assertionTypes' failedTemplate

## How to run locally?
* `helm unittest .`

## Notes
* `helm template .`
  * Problems:
    * Problem1: It does NOT throw error for 'configMap.yaml'
      * Solution: Comment 'validation.tpl' code and you get the same error as check in 'configMap_test.yaml'