# Enhanced message structure

Date: 01-08-2022

## Status

Accepted

## Context

The current message structure is very TDR-TRE specific. We would like a more generic structure for future extensibility.

## Decision

The following is now proposed for the message structures. This is based on the notion that messages:
- should be able to linked to see the thread. 
    - This is achieved through UUIDs
    - Every message in a chain adds a new UUID to a 
- should be actionable and allow a model driven approach allowing engines to act on them
    - Achieved by stating resource
- should define who is sending the message
    - achieved by creating a producer
- Allow data passed on to be identified and where they came from
    - achieved by creating parameter sections

### TDR and TRE interaction

The TDR to TRE message can be constructed as follows
TDR creates a UUID and states who has produced it  and why with any parameters 

```JSON
{
  "version" : "1.0.0",
  "timestamp" : "1659696719450929000",
  "UUIDs" : [
    { "TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"}
  ],
  "producer" : {
     "name" : "TDR",
     "process" : "consignment-export",
	   "type" : "judgment",
	   "environment" : "INT"
 },
 "parameters": {
  	"TDR" : {
      "resource": {
          "resource-type" : "Object",
          "access-type":"url",
          "value":"https://presignedurldemo.s3.eu-west2.amazonaws.com/TDR-2021-GW3.tar.gz........."
        },
        "resource-validation": {
          "resource-type" : "Object",
          "access-type" : "url",
          "validation-method" : "SHA256",
          "value" : "https://presignedurldemo.s3.eu-west2.amazonaws.com/TDR-2021-GW3.tar.gz.sha256........."
        },
        "number-of-retries" : "0",
        "reference" : "TDR-2021-GW3"
    }
  }
}
```

Should TDR need to send a retry the message would look like the following where TRE has added a new UUID.
TRE should also send any errors that caused the retry

```JSON
{
  "version" : "1.0.0",
  "timestamp" : "1659696719450929000",
  "UUIDs" : [
	  {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"},
	  {"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"}
  ],
  "producer" : {
    "name" : "TRE",
    "process" : "receive-and-validate-bag" ,
	  "type" : "judgment",
	  "environment" : "INT"
  },
  "parameters" : {
	  "TRE": {
      "number-of-retries" : "1",
      "reference" : "TDR-2021-GW3",
  		"errors" : [
  		    {"error" : "validation failed"},
  		    {"error" : "xxxx....."}
  		]
	  }
	}
}
```

And the response from TDR would be as

```JSON
{
  "version" : "1.0.0",
  "timestamp" : "1659696719450929000",
  "UUIDs" : [
    {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"},
	{"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"},
    {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfag3"}
  ],
  "producer" : {
     "name" : "TDR",
     "process" : "consignment-export",
	   "type" : "judgment",
	   "environment" : "INT"
 },
 "parameters": {
  	"TDR" : {
      "resource": {
          "resource-type" : "Object",
          "access-type":"url",
          "value":"https://presignedurldemo.s3.eu-west2.amazonaws.com/TDR-2021-GW3.tar.gz........."
        },
        "resource-validation": {
          "resource-type" : "Object",
          "access-type" : "url",
          "validation-method" : "SHA256",
          "value" : "https://presignedurldemo.s3.eu-west2.amazonaws.com/TDR-2021-GW3.tar.gz.sha256........."
        },
        "number-of-retries" : "1",
        "reference" : "TDR-2021-GW3"
    }
  }
}
```

### TRE to Access

```JSON
{
  "version" : "1.0.0",
  "timestamp" : "1659696719450929000",
  "UUIDs" : [
    {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"},
	{"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"},
    {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfag3"},
    {"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e454545454h4"}
  ],
  "producer" : {
    "name" : "TRE",
    "process" : "caselaw-export",
	  "type" : "Judgment",
	  "environment" : "INT"
  },
  "parameters": {
      "TRE": {
        "reference": "TDR-2021-GW3",
        "s3-folder-url": "https://prod-tre-editorial-judgment-out.s3.amazonaws.com/parsed/judgment/TDR-2021-GW3/0/0/TRE-TDR-2021-GW3.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256...",
        "s3-sha256-url": "https://prod-tre-editorial-judgment-out.s3.amazonaws.com/parsed/judgment/TDR-2021-GW3/0/0/TRE-TDR-2021-GW3.tar.gz.sha256?X-Amz-Algorithm=AWS4-HMAC-SHA256...",
        "consignment-type": "judgment",
        "number-of-retries": 0,
        "tar-gz": {
          "bucket": "prod-tre-editorial-judgment-out",
          "key": "parsed/judgment/TDR-2021-GW3/0/0/TRE-TDR-2021-GW3.tar.gz",
          "items": [
            {
              "name": "TDR-2021-GW3/TRE-TDR-2021-GW3-metadata.json",
              "size": 1459
            },
            {
              "name": "TDR-2021-GW3/TDR-2021-GW3.xml",
              "size": 118210
            },
            {
              "name": "TDR-2021-GW3/parser.log",
              "size": 1755
            },
            {
              "name": "TDR-2021-GW3/LC-2020-75 final.doc.docx",
              "size": 134498
            },
            {
              "name": "TDR-2021-GW3/image2.png",
              "size": 24744
            },
            {
              "name": "TDR-2021-GW3/image1.png",
              "size": 4957
            }
          ]
        },
        "prod-tre-version": "0.1.2",
        "payload": {
          "filename": "LC-2020-75 final.doc.docx",
          "xml": "TDR-2021-GW3.xml",
          "metadata": "TRE-TDR-2021-GW3-metadata.json",
          "images": [
            "image2.png",
            "image1.png"
          ],
          "log": "parser.log"
        },
        "lambda-functions-version": [
          {"prod-tre-bagit-checksum-validation": "0.0.9"},
          {"prod-tre-files-checksum-validation": "0.0.7"},
          {"prod-tre-prepare-parser-input": "0.0.19"},
          {"prod-tre-editorial-integration": "0.0.20"},
          {"prod-tre-run-judgment-parser": "v0.7.12"},
          {"prod-tre-slack-alerts": "0.0.24"}
        ]
      },
      "PARSER": {
        "uri": "https://caselaw.nationalarchives.gov.uk/...",
        "court": "UKUT-LC",
        "cite": "[2022] UKUT ...",
        "date": "2022-08-03",
        "name": "MR ...",
        "attachments": [],
        "error-messages": []
      },
      "TDR": {
        "Consignment-Type": "judgment",
        "Bag-Creator": "TDRExportv0.0.108",
        "Consignment-Start-Datetime": "2022-08-03T13:05:45Z",
        "Consignment-Series": "",
        "Source-Organization": "HM Courts and Tribunals Service",
        "Contact-Name": "Emily ...",
        "Internal-Sender-Identifier": "TDR-2021-GW3",
        "Consignment-Completed-Datetime": "2022-08-03T13:09:27Z",
        "Consignment-Export-Datetime": "2022-08-03T13:10:27Z",
        "Contact-Email": "test@judiciary.uk",
        "Payload-Oxum": "134498.1",
        "Bagging-Date": "2022-08-03"
      }
  }
}
```

Any errors should also be sent as this could be used to route the package more effectively. 
Note: just wondering if we should also add error type e.g. warning, critical 

If Access need to retry the message would be as

```JSON
{
	"version" : "1.0.0",
 	"timestamp" : "1659696719450929000",
	"UUIDs" : [
		{"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"},
		{"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"},
		{"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfag3"},
		{"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e454545454h4"},
		{"<name for caselaw>-UUID": "0094551e-a1fc-46c0-a6c7-e45465455455"}
	],
  "producer": {
       "name": "Access",
       "process": "caselaw-retry" ,
  	   "type": "Judgment",
       "environment" : "INT"
  },
  "parameters" : {
  	"TRE": {
  		 "number-of-retries" : "1",
  		 "reference" : "TRE-TDR-2021-GW3"
    },
  	"Access": {
  		"errors": [ 
  		  {"error": "validation failed"},
  		  {"error": "xxxx....."}
  		]
    }
  }
}
```

### TRE to DRI SIP

```JSON
{
  "version" : "1.0.0",
  "timestamp" : "1659696719450929000",
  "UUIDs" : [
    {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"},
    {"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"},
    {"TDR-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfag3"},
    {"TRE-UUID" : "0009885d-a1fc-46c0-a6c7-e454545454h4"}
  ],
  "producer" : {
    "name" : "TRE",
    "process" : "dri-sip-export",
    "type" : "standard",
    "environment" : "INT"
  },
  "parameters": {
      "TRE": {
        "reference": "TRE-TDR-2021-GW3",
        "s3-folder-url": "https://prod-tre-editorial-judgment-out.s3.amazonaws.com/parsed/judgment/TDR-2021-GW3/0/0/TRE-TDR-2021-GW3.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256...",
        "s3-sha256-url": "https://prod-tre-editorial-judgment-out.s3.amazonaws.com/parsed/judgment/TDR-2021-GW3/0/0/TRE-TDR-2021-GW3.tar.gz.sha256?X-Amz-Algorithm=AWS4-HMAC-SHA256...",
        "number-of-retries": 0,
        "prod-tre-version": "0.1.2",
        "lambda-functions-version": [
          {"prod-tre-bagit-checksum-validation": "0.0.9"},
          {"prod-tre-files-checksum-validation": "0.0.7"},
          {"prod-tre-prepare-parser-input": "0.0.19"},
          {"prod-tre-editorial-integration": "0.0.20"},
          {"prod-tre-run-judgment-parser": "v0.7.12"},
          {"prod-tre-slack-alerts": "0.0.24"}
        ]
      }
  }
}
```


## Consequences

The message could become quite big depending on the number of consumers and producers. 

