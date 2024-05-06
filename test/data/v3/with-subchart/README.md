* helm chart with [soft subcharts](https://helm.sh/docs/glossary/#chart-dependency-subcharts)
  * soft subchart
    * NOT installed via `helm dependency`
    * Here, you can find under `/charts`

## Notes
* If you have customized soft subchart 
  * -> by default, tests inside would also be executed
    * `helm unittest .` and check that child-chart's tests are executed!!
  * `helm unittest --with-subchart=false .` to skip running child-chart's tests
* Values defined in subchart tests are automatically scoped!!
  * == NOT need to prefix
  
    ```yaml
    # with-subchart/charts/child-chart/tests/xxx_test.yaml
    templates:
      - templates/xxx.yaml
    tests:
      - it:
        set:
          # no need to prefix with "child-chart."
          somevalue: should_be_scoped
        asserts:
          - ...
    ```