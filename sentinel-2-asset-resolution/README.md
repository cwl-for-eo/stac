# Sentinel-2 asset resolution 

This CWL document takes an URL to a Sentinel-2 STAC Item and resolves the asset key provided as input parameter.

It relies on `curl` and `jq` to get the STAC Item and parse its JSON content.

## The CWL document

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

It may be run with the parameters:

```yaml
stac_item: "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_53HPA_20210723_0_L2A"
asset: "B8A"
```

The execution will generate:

```console
INFO /srv/conda/bin/cwltool 3.0.20210319143721
INFO Resolved 'asset.cwl' to 'file:///home/fbrito/work/sentinel-2-dnbr-multitemporal-cog/asset.cwl'
INFO [job asset.cwl] /tmp/yy2yfe77$ docker \
    run \
    -i \
    --mount=type=bind,source=/tmp/yy2yfe77,target=/QUdWZm \
    --mount=type=bind,source=/tmp/ni40vjom,target=/tmp \
    --workdir=/QUdWZm \
    --read-only=true \
    --log-driver=none \
    --user=1000:1000 \
    --rm \
    --env=TMPDIR=/tmp \
    --env=HOME=/QUdWZm \
    --cidfile=/tmp/sqyngv0d/20210803093320-573822.cid \
    terradue/jq \
    /bin/sh \
    -c \
    'curl' '-s' 'https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_53HPA_20210723_0_L2A' | 'jq' '.assets.B8A.href' | 'tr' '-d' \" > /tmp/yy2yfe77/message
INFO [job asset.cwl] Max memory used: 0MiB
INFO [job asset.cwl] completed success
{
    "asset_href": "https://sentinel-cogs.s3.us-west-2.amazonaws.com/sentinel-s2-l2a-cogs/53/H/PA/2021/7/S2B_53HPA_20210723_0_L2A/B8A.tif"
}
INFO Final process status is success
```
