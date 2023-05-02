# Naming logic for TRE events and processes

Date: 17-11-2022

## Status

Accepted ___22-03-2023 This is now deprecated see this [repository](https://github.com/nationalarchives/da-transform-schemas/tree/main/tre_schemas) for the latest naming of TRE events and processes___.

## Context

There was a need to provide consistency around naming for TRE events and processes. A generic naming structure was needed to follow for current and future names.

## Decision

The below table shows agreed names for Events and Processes, some are for existing and some are for proposed future components. The table does not include an exhaustive list but provides a naming logic which should cover future naming requirements. Generally names should be simple and explain what is happening, e.g. when a new package is available it should be "name-of-package" and then "-available". The input and output of events listed below are illustratative and not comprehensive. It was discussed that for retry events if the retry is just asking for new pre-signed url links then this should be captured by running as a named retry event e.g. bag-available-retry, if the retry is re-running the whole process it will use the normal process name.

It was also discussed that in the future status notifications could be recieved from other services this could use the nomenclature of '-status' e.g. 'dri-status'.
<br>
<br>

| Current Process Name | Suggested Process Name | Purpose | Current Response Event(s) | Suggested Response Event Name(s) | Inputs | Outputs | Current Raised Event | Suggested Success Event Name | Error Event Name | Retry Event Name | Status Event Name
|---|---|---|---|---|---|---|---|---|---|---|---|
| validate-bagit | validate-bag | To pull a Bag from a S3 signed url checks and unzips a bag if required then validates that a bag is complete and that all checksums match | bagit-available | bag-available | - S3 Signed url for bag and sha256 <br> - Reference <br> - BagType | - S3 Location of the validated Bag directory <br> - BagType <br> - Reference | bagit-validated | validated-bag-available | validated-bag-error (e.g if a checksum error is identified an error is sent back to TDR) | bag-available-retry (e.g. if a new pre-signed URL is needed for a reason other than an error) |  |
| tre-judgment-router-process | get-bag-judgment | Find locate judgment and attachments from a Bag | bagit-validated AND type Judgment | validated-bag-available AND type Judgment | - S3 Location of the validated Bag directory <br>- BagType <br> - Reference | - S3 Location of the judgment and any attachments <br> - Reference | run-judgment-parser | judgment-available | judgment-available-error (e.g. unable to identify bag location) |  |  |
| tre-jp | run-judgment-parser | To take a judgment docx from a location and run the parser against it | run-judgment-parser | judgment-available | - S3 Location of the judgment and any attachments <br> - Reference | - S3 Location of the judgment and any attachments and parser output docs <br> - Parsed judgment fields - court, cite, date, name <br> - Reference | parsed-judgment-available | parsed-judgment-available | parsed-judgment-error (e.g parser unable to run) |  |  |
| create-caselaw-output | create-judgment-package | To take a parsed judgment output and create a output package for Caselaw | parsed-judgment-available | parsed-judgment-available | - S3 Location of the judgment and any attachments and parser output docs <br> - Parsed judgment fields - court, cite, date, name <br> - Reference | - S3 signed URL of the caselaw gzip output (including judgment, attachments and any parser output docs, metadata.json, bag-info.text) <br> - S3 signed URL of sha256 of gzip output) <br> -Parsed judgment fields - court, cite, date, name <br> - Reference | cl-package-out | judgment-package-available | judgment-package-error (e.g. unable to create package) |  |  |
| run-dri-pre-ingest-sip | create-dri-pre-ingest-sip | To create a SIP package for DRI for any standard transfer or judgment which requires preservation | bagit-validated AND type standard OR parsed-judgment-available | validated-bag-available AND type standard OR parsed-judgment-available AND TDR consignment ref | - S3 Location of the validated Bag directory <br> - BagType <br> - Reference <br> (If judgment - court, cite, date, name fields.) | - S3 signed URL's of SIP gzip output (files and folders, metadata.csv, closure.csv, closure.csvs, metadata.csvs, closure.csv.sha256, metadata.csv.sha256) <br> - S3 signed URL's of sha256 of gzip package | dri-preingest-sip-available | dri-preingest-sip-available | dri-preingest-sip-error (e.g. unable to create sip) | create-dri-pre-ingest-sip (e.g. DRI requests to re-run sip generation process for something they have already received) |dri-status  (e.g. message notifying of current status of records in DRI, such as that they are now ingested and being preserved) |
