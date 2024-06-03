* Validate if good default values are accidentally overwritten within your parent helm chart

# Soft subcharts
* [soft subcharts](https://helm.sh/docs/glossary/#chart-dependency-subcharts)
  * characteristics
      * NOT installed via `helm dependency`
      * Here, you can find under `/charts`
* About tests 
  * by default, tests inside would also be executed
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
# Hard subcharts
* [hard subcharts](https://helm.sh/docs/glossary/#chart-dependency-subcharts)
  * characteristics
    * existed already in `charts` directory
      * they should have been installed -- via -- `helm dependency build`
* About tests
  * NOT packaged 
  * `helm unittest .` to unittest these from the root chart
    * check how 'tests/postgresql_secrets_test.yaml' & 'tests/postgresql_deployment_test.yaml' are executed

```yaml
# $YOUR_CHART/tests/xxx_test.yaml
templates:
  - charts/postgresql/templates/xxx.yaml
tests:
  - it:
    set:
      # this time required to prefix with "postgresql."
      postgresql.somevalue: should_be_scoped
    asserts:
      - ...
```
# Notes
* assertType's `equal.decodeBase64` -- Check 'postgresql_secrets_test.yaml' --