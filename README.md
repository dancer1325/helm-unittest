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
- [Dependent subchart Testing](#dependent-subchart-testing)
- [Tests within subchart](#tests-within-subchart)
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

The docker container contains the fully installed helm client, including the helm-unittest plugin.

## Get Started

Add `tests` in `.helmignore` of your chart, and create the following test file at `$YOUR_CHART/tests/deployment_test.yaml`:

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

Now there is your first test! ;)

## Test Suite File

The test suite file is written in pure YAML, and default placed under the `tests/` directory of the chart with suffix `_test.yaml`. You can also have your own suite files arrangement with `-f, --file` option of cli set as the glob patterns of test suite files related to chart directory, like:

```bash
$ helm unittest -f 'my-tests/*.yaml' -f 'more-tests/**/*.yaml' my-chart
```

Check [DOCUMENT](./DOCUMENT.md) for more details about writing tests.

### Templated Test Suites

You may find yourself needing to set up a lots o tests that are a parameterization of a single test. For instance, let's say that you deploy to 3 environments `env = dev | staging | prod`.

In order to do this, you can actually write your tests as a helm chart as well. If you go about this route, you
must set the `--chart-tests-path` option. Once you have done so, helm unittest will run a standard helm render
against the values.yaml in your specified directory.

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

In the above example file structure, you would maintain a helm chart that will render out against the Chart.yaml
that as provided and the values.yaml. With rendered charts, any test suite that is generated is automatically ran
we do not look for a file postfix or glob.

**Note:** since you can create multiple suites in a single template file, you must provide the suite name, since we can no longer use the test suite file name meaningfully.

**Note 2:** since you can be running against subcharts and multiple charts, you need to make sure that you do not designate your `--chart-tests-path` to be the same folder as your other tests. This is because we will try to render those non-helm test folders and fail during the unit test.

**Note 3:** for snapshot tests, you will need to provide a helm ignore that ignores `*/__snapshot__/*`. Otherwise, subsequent runs will try to render those snapshots.

The command for the above chart and test configuration would be:

```shell
helm unittest --chart-tests-path tests-chart my-chart
```

## Usage

```
$ helm unittest [flags] CHART [...]
```

This renders your charts locally (without tiller) and runs tests
defined in test suite files.

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

### Yaml JsonPath Support

Now JsonPath is supported for mappings and arrays.
This makes it possible to find items in an array, based on JsonPath.
For more detail on the [`jsonPath`](https://github.com/vmware-labs/yaml-jsonpath#syntax) syntax.

Due to the change to JsonPath, the map keys in `path` containing periods (`.`) or special characters (`/`) are now supported with the use of `""`:

```yaml
- equal:
    path: metadata.annotations["kubernetes.io/ingress.class"]
    value: nginx
```

The next releases it will be possible to validate multiple paths when JsonPath result into multiple results.

### DocumentSelector

The test job or assertion can also specify a documentSelector rather than a documentIndex. Note that the documentSelector will always override a documentIndex if a match is found. This field is particularly useful when helm produces multiple templates and the order is not always guaranteed.

The `path` in the documentSelector has Yaml JsonPath Support, using JsonPath expressions it is possible to filter on multiple fields.

The `value` in the documentSelector can validate complete yaml objects.

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

## Example

Check [`test/data/v3/basic/`](./test/data/v3/basic) for some basic use cases of a simple chart.

## Snapshot Testing

Sometimes you may just want to keep the rendered manifest not changed between changes without every details asserted. That's the reason for snapshot testing! Check the tests below:

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

The `matchSnapshot` assertion validate the content rendered the same as cached last time. It fails if the content changed, and you should check and update the cache with `-u, --update-snapshot` option of cli.

```
$ helm unittest -u my-chart
```

The cache files is stored as `__snapshot__/*_test.yaml.snap` at the directory your test file placed, you should add them in version control with your chart.

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
