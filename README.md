---
title: Report
---
# Scribe GitHub actions - `valint report`
Scribe offers GitHub actions for embedding evidence collecting and integrity verification to your workflows. \
Action 

## Other actions
* [bom - action](https://github.com/scribe-security/action-bom/README.md)
* [verify - action](https://github.com/scribe-security/action-verify/README.md)
* [integrity report - action](https://github.com/scribe-security/action-report/README.md)
* [installer - action](https://github.com/scribe-security/action-installer/README.md)

## Report action
Action for `valint report`.
Once a set of evidence is uploaded to Scribe service an integrity report is generated on your build.
At the end of your pipeline run, decide to accept or fail a build, depending on the integrity analysis result reported by Scribe.  

### Input arguments
```yaml
  verbose:
    description: 'Increase verbosity (-v = info, -vv = debug)'
    default: 1
  config:
    description: 'Application config file'
  output-directory:
    description: 'Output directory path'
    default: ./scribe/valint
  output-file:
    description: 'Output file path'
  scribe-enable:
    description: 'Enable scribe client'
    default: false
  scribe-client-id:
    description: 'Scribe client id' 
  scribe-client-secret:
    description: 'Scribe access token' 
  scribe-url:
    description: 'Scribe url' 
  scribe-login-url:
    description: 'Scribe auth login url' 
  scribe-audience:
    description: 'Scribe auth audience' 
  context-dir:
    description: 'Context dir' 
  section:
    description: 'Select report sections'
  integrity:
    description: 'Select report integrity'
```

### Output arguments
```yaml
  output-file:
    description: 'Report output file path'
```

### Usage
```YAML
- name: Valint - download report
  id: valint_report
  uses: scribe-security/actions/valint/report@master
  with:
      verbose: 2
      scribe-enable: true
      product-key: ${{ secrets.product-key }}
      scribe-client-id: ${{ secrets.client-id }}
      scribe-client-secret: ${{ secrets.client-secret }}
```

# Integrations
## Scribe service integration
Scribe provides a set of services to store, verify and manage the supply chain integrity. \
Following are some integration examples.

Scribe integrity flow - upload evidence using `valint` and download the integrity report using `valint`. \
You may collect evidence anywhere in your workflows.

<details>
  <summary>  Scribe integrity report - full workflow </summary>

Full workflow example of a workflow, upload evidence and download report using Valint.

```YAML
name: example workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-report-test:
    runs-on: ubuntu-latest
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - uses: actions/checkout@v3
        with:
          repository: mongo-express/mongo-express
          ref: refs/tags/v1.0.0-alpha.4
          path: mongo-express-scm

      - name: valint Scm generate bom, upload to scribe
        id: valint_bom_scm
        uses: scribe-security/action-bom@master
        with:
           type: dir
           target: 'mongo-express-scm'
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - name: Build and push remote
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: mongo-express:1.0.0-alpha.4

      - name: valint Image generate bom, upload to scribe
        id: valint_bom_image
        uses: scribe-security/action-bom@master
        with:
           target: 'mongo-express:1.0.0-alpha.4'
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - name: Valint - download report
        id: valint_report
        uses: scribe-security/actions/valint/report@master
        with:
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - uses: actions/upload-artifact@v2
        with:
          name: scribe-reports
          path: |
            ${{ steps.valint_bom_scm.outputs.OUTPUT_PATH }}
            ${{ steps.valint_bom_image.outputs.OUTPUT_PATH }}
            ${{ steps.valint_report.outputs.OUTPUT_PATH }}
```
</details>


<details>
  <summary>  Scribe integrity report - Multi workflow </summary>

Full workflow example of a workflow, upload evidence and download report using valint

```YAML
name: example workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-report-test:
    runs-on: ubuntu-latest
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - uses: actions/checkout@v3
        with:
          repository: mongo-express/mongo-express
          ref: refs/tags/v1.0.0-alpha.4
          path: mongo-express-scm

      - name: Build and push remote
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: mongo-express:1.0.0-alpha.4

      - name: valint Image generate bom, upload to scribe
        id: valint_bom_image
        uses: scribe-security/action-bom@master
        with:
           target: 'mongo-express:1.0.0-alpha.4'
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - name: Valint - download report
        id: valint_report
        uses: scribe-security/actions/valint/report@master
        with:
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - uses: actions/upload-artifact@v2
        with:
          name: scribe-reports
          path: |
            ${{ steps.valint_bom_scm.outputs.OUTPUT_PATH }}
            ${{ steps.valint_bom_image.outputs.OUTPUT_PATH }}
            ${{ steps.valint_report.outputs.OUTPUT_PATH }}
```
</details>

## Integrity report examples
<details>
  <summary>  Scribe integrity report </summary>

Valint downloading integrity report from scribe service

```YAML
  - name: Valint - download report
    id: valint_report
    uses: scribe-security/actions/valint/report@master
    with:
        verbose: 2
        scribe-enable: true
        product-key:  ${{ secrets.product-key }}
        scribe-client-id: ${{ secrets.client-id }}
        scribe-client-secret: ${{ secrets.client-secret }}
```
</details>

<details>
  <summary>  Scribe integrity report, select section </summary>

Valint downloading integrity report from scribe service

```YAML
  - name: Valint - download report
    id: valint_report
    uses: scribe-security/actions/valint/report@master
    with:
        verbose: 2
        scribe-enable: true
        product-key:  ${{ secrets.product-key }}
        scribe-client-id: ${{ secrets.client-id }}
        scribe-client-secret: ${{ secrets.client-secret }}
        section: packages
```
</details>

<details>
  <summary> Install valint (tool) </summary>

Install valint as a tool
```YAML
- name: install valint
  uses: scribe-security/actions/valint/installer@master

- name: valint run
  run: |
    valint --version
    valint bom busybox:latest -vv
    valint report --scribe.client-id $SCRIBE_CLIENT_ID $SCRIBE_CLIENT_SECRET

``` 
</details>
