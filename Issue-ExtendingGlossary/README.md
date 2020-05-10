# Extending Atlas Glossay Terms #

## Dependancies ##

- Terminal with `${ATLAS_HOST}` variable set to the hostname of Atlas
- CLI tool `jq` installed

## Steps to Reproduce ##

1. Get the guid of `AtlasGlossaryTerm` guid

```sh
curl -u admin:admin -X GET -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/types/typedef/name/AtlasGlossaryTerm | jq '.guid'
```

Insert this value into 2-UpdateTerm.json as the value for `guid`

2. Create a new relationship

```sh
curl -u admin:admin -X POST -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/types/typedefs -d @1-NewRelationship.json | jq '.'
```

3. Update the `AtlasGlossaryTerm`

```sh
curl -u admin:admin -X PUT -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/types/typedefs -d @2-UpdateTerm.json | jq '.'
```

> At this point, the Term should allow for new **Related Terms** called _test_. Thius does not appear in the UI so I have added manual steps to show that it doesnt assciate via the API either.

As a **Related Term** is just a set of relationships (see below) we will test by creating these features.

```json
{
  "guid": "e291e60a-ed0d-4077-8e27-a902c945b525",
  "qualifiedName": "testterm@glossary",
  "name": "testterm",
  "anchor": {
    "glossaryGuid": "2cf6be04-7392-449f-8b20-a9a551662784",
    "relationGuid": "f5396040-673f-4a4d-977e-a7070afa2a05"
  },
  "seeAlso": [
    {
      "termGuid": "ac113632-c386-445a-84b9-753b0104aafc",
      "relationGuid": "45c6598d-4572-49f9-a772-d8a4b312861c",
      "displayText": "testterm2"
    }
  ]
}
```

4. Create a Glossary and 2 terms in Atlas. Then run the following command to get the GUIDs

```sh
curl -u admin:admin -X GET -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/glossary | jq '.[].terms[].termGuid'
```

5. Modify the file `3-ManualRelationship.json` for the term guids then run the following command.

```sh
relationshipGuid=`curl -u admin:admin -X POST -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/relationship -d @3-ManualRelationship.json | jq '.guid'`
```

6. Select a term (one of the guids) and get the details by the following command

```sh
 curl -u admin:admin -X GET -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/glossary/term/<insert_term_guid_here> | jq '.'
```

Now replace the contents of the file `4-ManualAppending.json` with the output and append the association _test_ like below where _termGuid_, _relationshipGuid_ and _displayText_ is equal to the output of the following query

```json
{
    "guid": "ac113632-c386-445a-84b9-753b0104aafc",
    "qualifiedName": "terma@testA",
    "name": "terma",
    "anchor": {
        "glossaryGuid": "2cf6be04-7392-449f-8b20-a9a551662784",
        "relationGuid": "cf55296d-98e7-4765-846c-2ea005d8e9b3"
    },
    "test": [
        {
            "termGuid": "e291e60a-ed0d-4077-8e27-a902c945b525",
            "relationGuid": "45c6598d-4572-49f9-a772-d8a4b312861c",
            "displayText": "termb"
        }
    ]
}
```

termGuid: the guid of the second term
relationshipGuid: `echo $relationshipGuid`
displaytext:

```sh
curl -u admin:admin http://${ATLAS_HOST}:21000/api/atlas/v2/glossary/term/<termGuid> | jq '.name'
```

Finally, to submit the appending, run the following command

```sh
curl -u admin:admin -X PUT -H "Content-type: application/json" http://${ATLAS_HOST}:21000/api/atlas/v2/glossary/term/ac113632-c386-445a-84b9-753b0104aafc -d @4-ManualAppending.json | jq '.'
```

## Outcome ##

You will see that the set for _test_ was not appended even though the `AtlasGlossaryTerm` supports it.