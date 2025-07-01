# GitLeaks Port Demo

## Blueprint

```
{
  "identifier": "leaked_secret",
  "description": "Leaked secrets",
  "title": "Leaked Secret",
  "icon": "Key",
  "schema": {
    "properties": {
      "commit": {
        "type": "string",
        "title": "Commit"
      },
      "commit_msg": {
        "type": "string",
        "title": "Commit Message"
      },
      "author": {
        "type": "string",
        "title": "Author"
      },
      "files": {
        "items": {
          "type": "string"
        },
        "type": "array",
        "title": "Files"
      },
      "urls": {
        "icon": "DefaultProperty",
        "type": "array",
        "title": "URLs",
        "items": {
          "type": "string",
          "format": "url"
        }
      },
      "locations": {
        "items": {
          "type": "object"
        },
        "type": "array",
        "title": "Locations"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {
    "service": {
      "title": "Service",
      "target": "service",
      "required": false,
      "many": false
    },
    "pull_request": {
      "title": "Pull Request",
      "target": "githubPullRequest",
      "required": false,
      "many": false
    },
    "user": {
      "title": "User",
      "target": "_user",
      "required": false,
      "many": false
    }
  }
}
```


## Webhook Mapping

```
[
  {
    "blueprint": "leaked_secret",
    "operation": "create",
    "filter": "true",
    "itemsToParse": ".body.runs[0].results",
    "entity": {
      "identifier": ".item.ruleId + ':' + .item.partialFingerprints.commitSha",
      "title": ".item.message.text",
      "properties": {
        "commit": ".item.partialFingerprints.commitSha",
        "author": ".item.partialFingerprints.author",
        "commit_msg": ".item.partialFingerprints.commitMessage",
        "locations": ".item.locations",
        "files": ".item.locations[].physicalLocation.artifactLocation.uri",
        "urls": ". as $a | .item as $b | .item.locations[] | \"https://github.com/\\($a.queryParams.repo)/blob/\\($b.partialFingerprints.commitSha)/\\(.physicalLocation.artifactLocation.uri)#L\\(.physicalLocation.region.startLine)\""
      },
      "relations": {
        "user": {
          "combinator": "'and'",
          "rules": [
            {
              "properties": "'$identifier'",
              "operator": "'='",
              "value": ".item.partialFingerprints.email"
            }
          ]
        },
        "service": ".queryParams.repo | split(\"/\") | last"
      }
    }
  }
]
```