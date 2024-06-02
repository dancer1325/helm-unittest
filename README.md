# helm unittest

[![CircleCI](https://circleci.com/gh/helm-unittest/helm-unittest.svg?style=svg)](https://circleci.com/gh/helm-unittest/helm-unittest)
[![Go Report Card](https://goreportcard.com/badge/github.com/helm-unittest/helm-unittest)](https://goreportcard.com/report/github.com/helm-unittest/helm-unittest)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=helm-unittest_helm-unittest&metric=alert_status)](https://sonarcloud.io/dashboard?id=helm-unittest_helm-unittest)

Unit test for _helm chart_ in YAML to keep your chart consistent and robust!

Feature:

- write test file in pure YAML
- **NOTHING** created /installed on your cluster
- [test suite](./DOCUMENT.md#test-suite)
- [test job](./DOCUMENT.md#test-job)
- [snapshot testing](#snapshot-testing)
- [test suite code completion and validation](#test-suite-code-completion-and-validation)

## Documentation

If you are ready for writing tests, check the [DOCUMENT](./DOCUMENT.md) for the test API in YAML.

- [Install](#install)
- [Docker Usage](#docker-usage)
- [Get Started](#get-started)
- [Test Suite File](#test-suite-file)
  - [Templated Test Suites](#templated-test-suites)
- [Usage](#usage)
  - [Flags](#flags)
  - [Yaml JsonPath Support](#yaml-jsonpath-support)
  - [DocumentSelector](#documentselector)
- [Example](#example)
- [Snapshot Testing](#snapshot-testing)
- [Hard / Dependent subchart Testing](#dependent-subchart-testing)
- [Soft / Tests within subchart](#tests-within-subchart)
- [Test suite code completion and validation](#test-suite-code-completion-and-validation)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Related Projects / Commands](#related-projects--commands)
- [Contributing](#contributing)

## Install

```
$ helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

It will install the latest version of binary into helm plugin directory.

## Docker Usage

```
# run help of latest helm with latest helm unittest plugin
docker run -ti --rm -v $(pwd):/apps helmunittest/helm-unittest

# run help of specific helm version with specific helm unittest plugin version
docker run -ti --rm -v $(pwd):/apps helmunittest/helm-unittest:3.11.1-0.3.0

# run unittests of a helm 3 chart
# make sure to mount local folder to /apps in container
docker run -ti --rm -v $(pwd):/apps helmunittest/helm-unittest:3.11.1-0.3.0 .

# run unittests of a helm 3 chart with Junit output for CI validation
# make sure to mount local folder to /apps in container
# the test-output.xml will be available in the local folder.
docker run -ti --rm -v $(pwd):/apps helmunittest/helm-unittest:3.11.1-0.3.0 -o test-output.xml -t junit .
```

* Docker container with
  * fully installed helm client + helm-unittest

## Get Started
* Test file folder must be included in `.helmignore` of your chart!!!
  * _Example:_ [prometheusChart](https://github.com/prometheus-community/helm-charts/blob/main/charts/alertmanager/.helmignore#L25)
* Add your first test file at `$YOUR_CHART/tests/deployment_test.yaml`:

```yaml
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should work
    set:
      image.tag: latest
    asserts:
      - isKind:
          of: Deployment
      - matchRegex:
          path: metadata.name
          pattern: -my-chart$
      - equal:
          path: spec.template.spec.containers[0].image
          value: nginx:latest
```

and run:

```
$ helm unittest $YOUR_CHART
```


## Test Suite File

* Pure YAML /
  * `tests/` is the default place
  * `_test.yaml` suffix
  * `-f pathToTheTests` /  `--file pathToTheTests` to specify nonDefault paths and names

    ```bash
    $ helm unittest -f 'my-tests/*.yaml' -f 'more-tests/**/*.yaml' my-chart
    ```
* Check [DOCUMENT](./DOCUMENT.md)

### Templated Test Suites

* Uses
  * Tests / are a parameterization of a single test
    * *Example:* Different environments `env = dev | staging | prod`
* == write tests as a helm chart
  * `helm unittest --chart-tests-path pathToTemplatedTestSuites ...`
    * `pathToTemplatedTestSuites` != other tests path
  * allows
    * creating multiple suites | 1! template file
      * suite name must be provided
* Add helm ignore rule `*/__snapshot__/*`
  * Otherwise, subsequent runs will try to render those snapshots
* Typical structure
```
/my-chart
  /tests-chart
    /Chart.yaml
    /values.yaml
    /templates
      /per_env_snapshots.yaml

  /Chart.yaml
  /values.yaml
  /.helmignore
  /templates
    /actual_template.yaml
```
Command to run

```shell
helm unittest --chart-tests-path tests-chart my-chart
```
* _Example:_ Check 'test/data/v3/with-helm-tests'

## Usage

```
$ helm unittest [flags] chartPath [...]
```
* How does it work?
  * Renders your charts locally &
    * Check 'test/data/v3/basic'
  * runs tests / defined in test suite files

### Flags

```
      --color                  enforce printing colored output even stdout is not a tty. Set to false to disable color
      --strict                 strict parse the testsuites (default false)
  -d  --debugPlugin            enable debug logging (default false)
  -v, --values stringArray     absolute or glob paths of values files location to override helmchart values
  -f, --file stringArray       glob paths of test files location, default to tests\*_test.yaml (default [tests\*_test.yaml])
  -q, --failfast               direct quit testing, when a test is failed (default false)
  -h, --help                   help for unittest
  -t, --output-type string     the file-format where testresults are written in, accepted types are (JUnit, NUnit, XUnit) (default XUnit)
  -o, --output-file string     the file where testresults are written in format specified, defaults no output is written to file
  -u, --update-snapshot        update the snapshot cached if needed, make sure you review the change before update
  -s, --with-subchart charts   include tests of the subcharts within charts folder (default true)
      --chart-tests-path string the folder location relative to the chart where a helm chart to render test suites is located
```

### Yaml/ JsonPath Support

* Check [`jsonPath`](https://github.com/vmware-labs/yaml-jsonpath#syntax)
* supported for
  * mappings &
  * arrays
* allows
  * making easier to find items in 
    * array or
    * map
* _Example:_ Check 'test/data/v3/basic'

```yaml
- equal:
    path: metadata.annotations["kubernetes.io/ingress.class"]
    value: nginx
```


### `documentSelector`

* Available for
  * test job &
  * assertion
* rules
  * if `documentSelector` & `documentIndex` match -> `documentSelector` takes precedence
* uses
  * multiple templates & 
  * order NOT always guaranteed
* Check [DOCUMENT.md](./DOCUMENT.md)


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
    documentSelector:
      path: metadata.name
      value: my-service-name
    asserts:
      - equal:
          path: metadata.name
          value: my-deploy
```

## Examples

* Check [`test/data/v3/`](./test/data/v3/basic)

## Snapshot Testing

* allows
  * storing the rendered manifests
    * cache files are stored as `__snapshot__/*_test.yaml.snap`
    * recommended to push to version control with your chart -- [Example](https://github.com/prometheus-community/helm-charts/tree/main/charts/alertmanager/unittests/__snapshot__) --
  * validating the current content rendered vs cached last time
* `matchSnapshot`
  * if it fails == content changed -> you should
    * check and
    * update the cache -- `helm unittest -u ...` / `helm unittest --update-snapshot ...` -- 
```yaml
templates:
  - templates/deployment.yaml
tests:
  - it: pod spec should match snapshot
    asserts:
      - matchSnapshot:
          path: spec.template.spec
  # or you can snapshot the whole manifest
  - it: manifest should match snapshot
    asserts:
      - matchSnapshot: {}
```


## Hard subchart Testing
* Check [`test/data/v3/with-subchart/`](./test/data/v3/with-subchart) as an example.

## Soft Subchart Testing
* Check [`test/data/v3/with-subchart/`](./test/data/v3/with-subchart) as an example.

## Test Suite code completion and validation

Most popular IDEs (IntelliJ, Visual Studio Code, etc.) support applying schemas to YAML files using a JSON Schema. This provides comprehensive documentation as well as code completion while editing the test-suite file:

![Code completion](./.images/testsuite-yaml-codecompletion.png)

In addition, test-suite files can be validated while editing so wrongfully added additional properties or incorrect data types can be detected while editing:

![Code Validation](./.images/testsuite-yaml-codevalidation.png)

### Visual Studio Code

When developing with VSCode, the very popular YAML plug-in (created by RedHat) allows adding references to schemas by adding a comment line on top of the file:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/helm-unittest/helm-unittest/main/schema/helm-testsuite.json
suite: http-service.configmap_test.yaml
templates: [configmap.yaml]
release:
  name: test-release
  namespace: TEST_NAMESPACE
```

Alternatively, you can add the schema globally to the IDE, using a well defined pattern:

```json
"yaml.schemas": {
  "https://raw.githubusercontent.com/helm-unittest/helm-unittest/main/schema/helm-testsuite.json": ["charts/*/tests/*_test.yaml"]
}
```

### IntelliJ

Similar to VSCode, IntelliJ allows mapping file patterns to schemas via preferences: Languages & Frameworks -> Schemas and DTDs -> JSON Schema Mappings

![Add Json Schema](./.images/testsuite-yaml-addschema-intellij.png)

## Frequently Asked Questions

As more people use the unittest plugin, more questions will come. Therefore a [Frequently Asked Question page](./FAQ.md) is created to answer the most common questions.

If you are missing an answer to a question, feel free to raise a ticket.

## Related Projects / Commands

This plugin is inspired by [helm-template](https://github.com/technosophos/helm-template), and the idea of snapshot testing and some printing format comes from [jest](https://github.com/facebook/jest).

And there are some other helm commands you might want to use:

- [`helm template`](https://github.com/kubernetes/helm/blob/master/docs/helm/helm_template.md): render the chart and print the output.

- [`helm lint`](https://github.com/kubernetes/helm/blob/master/docs/helm/helm_lint.md): examines a chart for possible issues, useful to validate chart dependencies.

- [`helm test`](https://github.com/kubernetes/helm/blob/master/docs/helm/helm_test.md): test a release with testing pod defined in chart. Note this does create resources on your cluster to verify if your release is correct. Check the [doc](https://github.com/kubernetes/helm/blob/master/docs/chart_tests.md).

Alternatively, you can also use generic tests frameworks:

- [Python](https://github.com/apache/airflow/issues/11657)

- Go - [terratest](https://blog.gruntwork.io/automated-testing-for-kubernetes-and-helm-charts-using-terratest-a4ddc4e67344)

## License

MIT

## Contributing

Issues and PRs are welcome!
Before start developing this plugin, you must have [Go](https://golang.org/doc/install) >= 1.21 installed, and run:

```
git clone git@github.com:helm-unittest/helm-unittest.git
cd helm-unittest
```

And please make CI passed when request a PR which would check following things:

- `gofmt` no changes needed. Please run `gofmt -w -s .` before you commit.
- `go test ./pkg/unittest/...` passed.

In some cases you might need to manually fix the tests in `*_test.go`. If the snapshot tests (of the plugin's test code) failed, you need to run:

```
UPDATE_SNAPSHOTS=true go test ./...
```

This update the snapshot cache file and please add them before you commit.
