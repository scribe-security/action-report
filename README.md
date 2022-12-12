---
title: Report
sidebar_position: 2
---
# Scribe GitHub Actions - `valint report`
Scribe offers GitHub Actions for embedding evidence collecting and validated integrity of your supply chain.

Use `valint report` to validate integrity of your supply chain.

Further documentation [Github integration](https://scribe-security.netlify.app/docs/ci-integrations/github/)


## Other Actions
* [bom - action](https://github.com/scribe-security/action-bom/README.md)
* [verify - action](https://github.com/scribe-security/action-verify/README.md)
* [integrity report - action](https://github.com/scribe-security/action-report/README.md)
* [installer - action](https://github.com/scribe-security/action-installer/README.md)

## Report Action
Action for `valint report`.
Once a set of evidence is uploaded to Scribe service an integrity report is generated on your build.
At the end of your pipeline run, decide to accept or fail a build, depending on the integrity analysis result reported by Scribe.  

### Input arguments
```yaml
  verbose:
    description: 'Log verbosity level [-v,--verbose=1] = info, [-vv,--verbose=2] = debug'
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
  OUTPUT_PATH:
    description: 'evidence output file path'
```

### Usage
```YAML
- name: Valint - download report
  id: valint_report
  uses: scribe-security/action-report@master
  with:
      verbose: 2
      scribe-enable: true
      product-key: ${{ secrets.product-key }}
      scribe-client-id: ${{ secrets.client-id }}
      scribe-client-secret: ${{ secrets.client-secret }}
```

## Integrations

### Before you begin
Further documentation [Github integration](https://scribe-security.netlify.app/docs/ci-integrations/github/)

### Usage
```yaml
- name: Download integrity report
  uses: scribe-security/action-report@master
  with:
      scribe-enable: true
      product-key:  ${{ secrets.product-key }}
      scribe-client-id: ${{ secrets.client-id }}
      scribe-client-secret: ${{ secrets.client-secret }}
```

### Scribe service integration

If you are using Github Actions as your Continuous Integration tool (CI), use these instructions to integrate Scribe into your workflows to protect your projects.

<details>
  <summary>  Scribe integrity report </summary>

Full workflow example of a workflow, upload evidence on source and image to Scribe. <br />
Download the integrity report,verifying the image integrity from Scribe.

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

      - name: gensbom Scm generate bom, upload to scribe
        id: gensbom_bom_scm
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

      - name: gensbom Image generate bom, upload to scribe
        id: gensbom_bom_image
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
        uses: scribe-security/action-report@master
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
            ${{ steps.gensbom_bom_scm.outputs.OUTPUT_PATH }}
            ${{ steps.gensbom_bom_image.outputs.OUTPUT_PATH }}
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
  uses: scribe-security/action-report@master
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
    uses: scribe-security/action-report@master
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
  <summary> Install Valint (tool) </summary>

Install Valint as a tool
```YAML
- name: install gensbom
  uses: scribe-security/action-installer@master
  with:
    tool: valint

- name: valint run
  run: |
    valint --version
    valint report --scribe.client-id $SCRIBE_CLIENT_ID $SCRIBE_CLIENT_SECRET
``` 
</details>
