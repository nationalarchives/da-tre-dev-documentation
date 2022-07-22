# Enhanced message structure

Date: 21-07-2022

## Status

Proposed

## Context

The current message structure is very TDR-TRE specific. We would like a more generic structure for future extensibility.

> Phil, anything else to add here?

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
TDR creates a UUID and states who produce it why and the parameters 

```JSON
{
  "UUIDs" : [
    { "Process-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"}
  ],
  "producer" : {
     "name" : "TDR",
     "process" : "export" ,
	 "type" : "Judgment",
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
  "UUIDs" : [
	  {"Process-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"} ,
	  {"Process-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"}
  ],
  "producer" : {
    "name" : "TRE",
    "process" : "pull retry" ,
	"type" : "Court Judgment",
	"environment" : "INT"
	 },
  "parameters" : {
	  "TDR": {
		 "number-of-retries" : "1",
		 "reference" : "TDR-2021-GW3"
		},
	  "TRE": {
		"errors" : [ 
		  {"error" : "validation failed"},
		  {"error" : "xxxx....."}
		]
	  }	
	}
}
```

> _Questions_: 
> - Why do we require multiple process-UUID? Which process-UUID would someone use to diagnose an issue using the message logs? This is the history section for the messsages, and UUID are unique for each message and everything touches this message should attach a new UUID if raising a new event.
> - Shall we remove the retry limit? What about when the retry limit is exceeded? Should TRE send another message? The limit is still valid, and the consumer should raise a new event informing the retry limit has been reached.

### TRE to Access

```JSON
{
  "UUIDs" : [
	  {"Process-UUID" : "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"} ,
	  {"Process-UUID" : "0009885d-a1fc-46c0-a6c7-e45454545455"}
  ],
  "producer" : {
    "name" : "TRE",
    "process" : "pull retry" ,
	"type" : "Court Judgment",
	"environment" : "INT"
	 },
  "parameters" : {
	  "TDR": {
		 "number-of-retries" : "1",
		 "reference" : "TDR-2021-GW3"
		},
	  "TRE": {
		"errors" : [ 
		  {"error" : "validation failed"},
		  {"error" : "xxxx....."}
		]
	  }	
	}
}
```

Any errors should also be sent as this could be used to route the package more effectively. 
Note: just wondering if we should also add error type e.g. warning, critical 

If Access need t retry the message would be as

```JSON
{
    "UUIDs": [
  	{"Process-UUID": "0001d50d-a1fc-46c0-a6c7-ebc8309cfaf2"},
  	{"Process-UUID": "0097675d-a1fc-46c0-a6c7-e45467696792"},
  	{"Process-UUID": "0094551e-a1fc-46c0-a6c7-e45465455455"}
  ],
  "producer": {
       "name": "Access",
       "process": "pull retry" ,
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


## Consequences

> What becomes easier or more difficult to do because of this change?
> Phil, anything to add here?