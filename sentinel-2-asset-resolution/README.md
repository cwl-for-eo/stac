# Sentinel-2 asset resolution 

This CWL document takes an URL to a Sentinel-2 STAC Item and resolves the asset key provided as input parameter.

It relies on `curl` and `jq` to get the STAC Item and parse its JSON content.

## CWL CommandLineTool 

The CWL document contains a `CommandLineTool` element that invoques the command:

```console
curl -s 'https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_53HPA_20210723_0_L2A' | jq '.assets.B8A.href' | tr -d '"'
```

The CWL document content is: 

```yaml
class: CommandLineTool
 
requirements:
  DockerRequirement: 
    dockerPull: terradue/jq
  ShellCommandRequirement: {}
  InlineJavascriptRequirement: {}

baseCommand: curl
arguments:
- -s
- valueFrom: ${ return inputs.stac_item; }
- "|"
- jq
- valueFrom: ${ return ".assets." + inputs.asset + ".href"; }
- "|"
- tr 
- -d
- '\"' #\""

stdout: message

inputs:
  stac_item:
    type: string
  asset:
    type: string

outputs:

  asset_href: 
    type: string
    outputBinding:
      glob: message
      loadContents: true
      outputEval: $( self[0].contents.split("\n").join("") )

cwlVersion: v1.0
```

## CWL Workflow

The `CommandLineTool` above is now included in a single step `Workflow`:

```yaml
$graph:
- class: Workflow
  label: Resolve STAC asset href
  doc: This workflow resolves a STAC asset href using its key 
  id: main

  inputs:
    stac_item:
      type: string
    asset:
      type: string

  outputs:
    asset_href:
      outputSource:
      - node_stac/asset_href
      type: string

  steps:

    node_stac:

      run: "#asset"

      in:
        stac_item: stac_item
        asset: asset

      out:
        - asset_href

- class: CommandLineTool
  id: asset

  requirements:
    DockerRequirement: 
      dockerPull: terradue/jq
    ShellCommandRequirement: {}
    InlineJavascriptRequirement: {}

  baseCommand: curl
  arguments:
  - -s
  - valueFrom: ${ return inputs.stac_item; }
  - "|"
  - jq
  - valueFrom: ${ return ".assets." + inputs.asset + ".href"; }
  - "|"
  - tr 
  - -d
  - '\"' #\""

  stdout: message

  inputs:
    stac_item:
      type: string
    asset:
      type: string

  outputs:

    asset_href: 
      type: string
      outputBinding:
        glob: message
        loadContents: true
        outputEval: $( self[0].contents.split("\n").join("") )

cwlVersion: v1.0
```


## Execution

It may be run with the parameters:

```yaml
stac_item: "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_53HPA_20210723_0_L2A"
asset: "B8A"
```

And: 

```console
cwltool asset.cwl asset.yml
```

The execution will generate:

```console
INFO /srv/conda/bin/cwltool 3.0.20210319143721
INFO Resolved 'asset.cwl' to 'file:///home/fbrito/work/stac/sentinel-2-asset-resolution/asset.cwl'
INFO [workflow ] start
INFO [workflow ] starting step node_stac
INFO [step node_stac] start
INFO [job node_stac] /tmp/baqarwll$ docker \
    run \
    -i \
    --mount=type=bind,source=/tmp/baqarwll,target=/mvOEKM \
    --mount=type=bind,source=/tmp/hc2s_c5k,target=/tmp \
    --workdir=/mvOEKM \
    --read-only=true \
    --log-driver=none \
    --user=1000:1000 \
    --rm \
    --env=TMPDIR=/tmp \
    --env=HOME=/mvOEKM \
    --cidfile=/tmp/quxi1nlw/20210803134014-592685.cid \
    terradue/jq \
    /bin/sh \
    -c \
    'curl' '-s' 'https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_53HPA_20210723_0_L2A' | 'jq' '.assets.B8A.href' | 'tr' '-d' \" > /tmp/baqarwll/message
INFO [job node_stac] Max memory used: 0MiB
INFO [job node_stac] completed success
INFO [step node_stac] completed success
INFO [workflow ] completed success
{
    "asset_href": "https://sentinel-cogs.s3.us-west-2.amazonaws.com/sentinel-s2-l2a-cogs/53/H/PA/2021/7/S2B_53HPA_20210723_0_L2A/B8A.tif"
}
INFO Final process status is success
```
