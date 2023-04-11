# Overall structure of SIP

- The transformations listed here relate to TDR bag output from 29/03/2023
- The payload files in bagit directory `data` are moved to SIP directory `batch/series/content`
- The field Consignment-Include-Top-Level-Folder is only used when the 'content' folder does not exist, behaviour is described below
- `closure.csv` and `metadata.csv` files are added to the `batch/series/` directory of the SIP following spec below (and they have a checksum file added alongside them, see sample data for example of files & their locations in SIP)
- `closure.csvs` and `metadata.csvs` files are copied to same dir (currently from hardcoded file)

# Fields from bagit used to make the SIP

| Bagit File            | Field                                | Example Value                      | Batch ID | closure.csv | metadata.csv |
| ---                   | ---                                  | ---                                | ---      | ---         | ---          |
|                       | file_type                            | File                               |          |    X        |       X      |
|                       | file_size                            | 70263                              |          |             |              |
| file-metadata.csv     | file_name                            | file-a1.txt                        |          |             |       X      |
|                       | clientside_original_filepath         | data/content/folder-a/file-a1.txt  |          |    X        |       X      |
|                       | rights_copyright                     | Crown Copyright                    |          |             |       X      |
|                       | legal_status                         | Public Record(s)                   |          |             |       X      |
|                       | held_by                              | The National Archives, Kew         |          |             |       X      |
|                       | date_last_modified                   | 2022-07-18T00:00:00                |          |             |       X      |
|                       | closure_type                         | Closed                             |          |    X        |              |
|                       | closure_start_date                   | 2022-07-18T12:44:52                |          |    X        |              |
|                       | closure_period                       | 50                                 |          |    X        |              |
|                       | foi_exemption_code                   | 27(1)                              |          |    X        |              |
|                       | foi_exemption_asserted               | 2022-07-18T00:00:00                |          |    X        |              |
|                       | title_closed                         | true                               |          |    X        |              |
|                       | title_alternate                      | file-[redacted].txt                |          |    X        |              |
|                       | description                          | This is a historic record          |          |             |       X      |
|                       | description_closed                   | true                               |          |    X        |              |
|                       | description_alternate                | This is a [redacted] record        |          |    X        |              |
|                       | language                             | English                            |          |             |       X      |
|                       | end_date                             | 2022-07-18T12:44:52                |          |             |       X      |
|                       | file_name_translaton                 | Translated File Name               |          |             |       X      |
|                       | original_filepath                    | data/content/folder-a/file-a1.txt  |          |             |       X      |
|                       | former_reference_department          | former/1                           |          |             |       X      |
| --- |
| manifest-sha256.txt   | first column                         | 4ef13 ... a34b                     |          |             |       X      |        
|                       | second column                        | data/content/folder-a/file-a1.txt  |          |             |              |        
| --- |
| bag-info.txt          | Consignment-Type                     | standard                           |          |             |              |        
|                       | Bag-Creator                          | TDRExportv0.0.205                  |          |             |              |        
|                       | Consignment-Start-Datetime           | 2022-07-18T12:39:41Z               |          |             |              |        
|                       | Consignment-Series                   | TSTA 1                             |    X     |    X        |       X      |        
|                       | Source-Organization                  | Testing A                          |          |             |              |        
|                       | Contact-Name                         | DA Example                         |          |             |              |        
|                       | Internal-Sender-Identifier           | TDR-2022-AA1                       |    X     |    X        |       X      |        
|                       | Consignment-Completed-Datetime       | 2022-07-18T12:44:52Z               |          |             |              |        
|                       | Consignment-Export-Datetime          | 2022-07-18T12:45:45Z               |          |             |       X      |        
|                       | Contact-Email                        | da.example@nationalarchives.gov.uk |          |             |              |        
|                       | Payload-Oxum                         | 3893083.18                         |          |             |              |        
|                       | Bagging-Date                         | 2022-07-18                         |          |             |              |        
|                       | Consignment-Include-Top-Level-Folder | false                              |          |    X        |       X      |        
# Constructing the SIP

## The Batch / Series make the path `batch/series`:

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |

When transform that bagit

Then dri batch is named `TSTA1Y22TBAA1`

Transformation in this example is:
+ `TSTA1` (Consignment-Series with the space removed)
+ `Y` 
+ `22` (2 digits from year Internal-Sender-Identifier)
+ `T`
+ `B` 
+ `AA1` (final "base 25" part of Internal-Sender-Identifier)

This batch name is used with the series name to construct the base directories in the sip: `batch/series` and as part 
of the `identifier` fields of `closure.csv` and `metadata.csv`. (The series name part of the path is simply the value 
from `bag-info.txt`'s `Consignment-Series` with the space removed)

# Constructing closure.csv / metadata.csv rows for a file

## The closure.csv row for a closed file with single exemption

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       |
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | file_type                               | File                               |
    | file-metadata.csv   | foi_exemption_code                      | 27(1)                              |
    | file-metadata.csv   | closure_type                            | Closed                             |
    | file-metadata.csv   | closure_start_date                      | 2022-07-18T12:44:52                |
    | file-metadata.csv   | closure_period                          | 50                                 |
    | file-metadata.csv   | foi_exemption_asserted                  | 2022-07-18T12:44:52                |
	| file-metadata.csv   | title_closed                            | true                               |
    | file-metadata.csv   | title_alternate                         | file-[redacted].txt                |
	| file-metadata.csv   | description_closed                      | true                               |
    | file-metadata.csv   | description_alternate                   | This is a [redacted] record        |

When transform that bagit

Then the bagit leads to closure.csv which has:

    | identifier             | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt          | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv filepath] 
    | folder                 | file                                                             | # transformed [file-metadata.csv file_type]
    | closure_type           | closed_for                                                       | # transformed [file-metadata.csv closure_type]
    | closure_start_date     | 2022-07-18T12:44:52                                              | # [file-metadata.csv closure_start_date]
    | closure_period         | 50                                                               | # [file-metadata.csv closure_period]
    | foi_exemption_code     | 27(1)                                                            | # [file-metadata.csv foi_exemption_code] 
    | foi_exemption_asserted | 2022-07-18T12:44:52                                              | # [file-metadata.csv foi_exemption_asserted]
    | title_public           | FALSE                                                            | # transformed [file-metadata.csv title_closed]
    | title_alternate        | file-[redacted].txt                                              | # [file-metadata.csv title_alternate]
    | description_public     | FALSE                                                            | # transformed [file-metadata.csv description_closed]
    | description_alternate  | This is a [redacted] record                                      | # [file-metadata.csv description_alternate]
	
Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" if a folder + special characters should be URI encoded
- file_type is transformed from 'File' to 'file'
- closure_type is transformed from 'Closed' to 'closed_for'
- title_public take logic from title_closed bagit field and flip it and capitalise, so 'true' becomes 'FALSE'
- description_public take logic from title_closed bagit field and flip it, so 'true' becomes 'FALSE'

## The closure.csv row for a closed file multiple exemptions

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       |
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | file_type                               | File                               |
    | file-metadata.csv   | foi_exemption_code                      | 27(1)|40(2)                        |
    | file-metadata.csv   | closure_type                            | Closed                             |
    | file-metadata.csv   | closure_start_date                      | 2022-07-18T12:44:52                |
    | file-metadata.csv   | closure_period                          | 50                                 |
    | file-metadata.csv   | foi_exemption_asserted                  | 2022-07-18T12:44:52                |
	| file-metadata.csv   | title_closed                            | false                              |
    | file-metadata.csv   | title_alternate                         |                                    |
	| file-metadata.csv   | description_closed                      | false                              |
    | file-metadata.csv   | description_alternate                   |                                    |

When transform that bagit

Then the bagit leads to closure.csv which has:

    | identifier             | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt          | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv filepath] 
    | folder                 | file                                                             | # transformed [file-metadata.csv file_type]
    | closure_type           | closed_for                                                       | # transformed [file-metadata.csv closure_type]
    | closure_start_date     | 2022-07-18T12:44:52                                              | # [file-metadata.csv closure_start_date]
    | closure_period         | 50                                                               | # [file-metadata.csv closure_period]
    | foi_exemption_code     | "27(1),40(2)"                                                    | # transformed [file-metadata.csv foi_exemption_code] 
    | foi_exemption_asserted | 2022-07-18T12:44:52                                              | # [file-metadata.csv foi_exemption_asserted]
    | title_public           | TRUE                                                             | # transformed [file-metadata.csv title_closed]
    | title_alternate        |                                                                  | # [file-metadata.csv title_alternate]
    | description_public     | TRUE                                                             | # transformed [file-metadata.csv description_closed]
    | description_alternate  |                                                                  | # [file-metadata.csv description_alternate]
	
Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" if a folder + special characters should be URI encoded
- file_type is transformed from 'File' to 'file'
- closure_type is transformed from 'Closed' to 'closed_for'
- for the foi_exemption_code bagit field all '|' symbols are replaced with a ','
- title_public take logic from title_closed bagit field and flip it and capitalise, so 'false' becomes 'TRUE'
- description_public take logic from title_closed bagit field and flip it, so 'false' becomes 'TRUE'

## The closure.csv row for a closed file with a description alternate and title alternate that includes a comma

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       |
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | file_type                               | File                               |
    | file-metadata.csv   | foi_exemption_code                      | 27(1)                              |
    | file-metadata.csv   | closure_type                            | Closed                             |
    | file-metadata.csv   | closure_start_date                      | 2022-07-18T12:44:52                |
    | file-metadata.csv   | closure_period                          | 50                                 |
    | file-metadata.csv   | foi_exemption_asserted                  | 2022-07-18T12:44:52                |
	| file-metadata.csv   | title_closed                            | true                               |
    | file-metadata.csv   | title_alternate                         | "alternate,title"                  |
	| file-metadata.csv   | description_closed                      | true                               |
    | file-metadata.csv   | description_alternate                   | "alternate,description"            |

When transform that bagit

Then the bagit leads to closure.csv which has:

    | identifier             | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt          | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv filepath] 
    | folder                 | file                                                             | # transformed [file-metadata.csv file_type]
    | closure_type           | closed_for                                                       | # transformed [file-metadata.csv closure_type]
    | closure_start_date     | 2022-07-18T12:44:52                                              | # [file-metadata.csv closure_start_date]
    | closure_period         | 50                                                               | # [file-metadata.csv closure_period]
    | foi_exemption_code     | 27(1)                                                            | # transformed [file-metadata.csv foi_exemption_code] 
    | foi_exemption_asserted | 2022-07-18T12:44:52                                              | # [file-metadata.csv foi_exemption_asserted]
    | title_public           | TRUE                                                             | # transformed [file-metadata.csv title_closed]
    | title_alternate        | "alternate,title"                                                | # transformed [file-metadata.csv title_alternate]
    | description_public     | TRUE                                                             | # transformed [file-metadata.csv description_closed]
    | description_alternate  | "alternate,description"                                          | # transformed [file-metadata.csv description_alternate]
	
Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" if a folder + special characters should be URI encoded
- file_type is transformed from 'File' to 'file'
- closure_type is transformed from 'Closed' to 'closed_for'
- for the foi_exemption_code bagit field all '|' symbols are replaced with a ','
- title_public take logic from title_closed bagit field and flip it and capitalise, so 'false' becomes 'TRUE'
- description_public take logic from title_closed bagit field and flip it, so 'false' becomes 'TRUE'
- description_alternate will be enclosed in "" if a comma is present
- title_alternate will be enclosed in "" if a comma is present

## The closure.csv row for an open file

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       |
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | file_type                               | File                               |
    | file-metadata.csv   | foi_exemption_code                      |                                    |
    | file-metadata.csv   | closure_type                            | Open                             	 |
    | file-metadata.csv   | closure_start_date                      |                                    |
    | file-metadata.csv   | closure_period                          |                                    |
    | file-metadata.csv   | foi_exemption_asserted                  |                                    |
	| file-metadata.csv   | title_closed                            | false                              |
    | file-metadata.csv   | title_alternate                         |                                    |
	| file-metadata.csv   | description_closed                      | false                              |
    | file-metadata.csv   | description_alternate                   |                                    |

When transform that bagit

Then the bagit leads to closure.csv which has:

	
    | identifier             | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt          | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv filepath] 
    | folder                 | file                                                             | # transformed [file-metadata.csv file_type]
    | closure_type           | open_on_transfer                                                 | # transformed [file-metadata.csv closure_type]
    | closure_start_date     |                                                                  | # empty
    | closure_period         | 0                                                                | # 0
    | foi_exemption_code     |                                                                  | # empty 
    | foi_exemption_asserted |                                                                  | # empty
    | title_public           | TRUE                                                             | # transformed [file-metadata.csv title_closed]
    | title_alternate        |                                                                  | # [file-metadata.csv title_alternate] 
    | description_public     | TRUE                                                             | # transformed [file-metadata.csv description_closed]
    | description_alternate  |                                                                  | # [file-metadata.csv description_alternate]

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" if a folder + special characters should be URI encoded
- remove capital of bagit field file_type to make dri field file
- closure_type is transformed from 'Open' to 'open-on-transfer'
- closure period is 0 if bag closure_type is Open
- title_public take logic from title_closed bagit field and flip it and capitalise, so 'false' becomes 'TRUE'
- description_public take logic from title_closed bagit field and flip it, so 'false' becomes 'TRUE'

## The closure.csv row for a folder

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       |
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/             |
    | file-metadata.csv   | file_type                               | Folder                             |
    | file-metadata.csv   | foi_exemption_code                      |                                    |
    | file-metadata.csv   | closure_type                            | Open                             	 |
    | file-metadata.csv   | closure_start_date                      |                                    |
    | file-metadata.csv   | closure_period                          |                                    |
    | file-metadata.csv   | foi_exemption_asserted                  |                                    |
	| file-metadata.csv   | title_closed                            | false                              |
    | file-metadata.csv   | title_alternate                         |                                    |
	| file-metadata.csv   | description_closed                      | false                              |
    | file-metadata.csv   | description_alternate                   |                                    |

When transform that bagit

Then the bagit leads to closure.csv which has:

	
    | identifier             | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/                     | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv filepath] 
    | folder                 | folder                                                           | # transformed [file-metadata.csv file_type]
    | closure_type           | open_on_transfer                                                 | # transformed [file-metadata.csv closure_type]
    | closure_start_date     |                                                                  | # empty
    | closure_period         | 0                                                                | # 0
    | foi_exemption_code     |                                                                  | # empty 
    | foi_exemption_asserted |                                                                  | # empty
    | title_public           | TRUE                                                             | # transformed [file-metadata.csv title_closed]
    | title_alternate        |                                                                  | # [file-metadata.csv title_alternate] 
    | description_public     | TRUE                                                             | # transformed [file-metadata.csv description_closed]
    | description_alternate  |                                                                  | # [file-metadata.csv description_alternate]

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" if a folder + special characters should be URI encoded
- remove capital of bagit field file_type to make dri field folder
- closure_type is transformed from 'Open' to 'open-on-transfer'
- closure period is 0 if bag closure_type is Open
- title_public take logic from title_closed bagit field and flip it and capitalise, so 'false' becomes 'TRUE'
- description_public take logic from title_closed bagit field and flip it, so 'false' becomes 'TRUE'

## The metadata.csv row for a file with mandatory metadata

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime             | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |

    | manifest-sha256.txt | first column                            | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                           | data/content/folder-a/file-a1.txt  |

    | file-metadata.csv   | file_name                               | file-a1.txt                        |
	| file-metadata.csv   | file_type                               | File                               |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | rights_copyright                        | Crown Copyright                    |
    | file-metadata.csv   | legal_status                            | Public Record(s)                   |
    | file-metadata.csv   | held_by                                 | "The National Archives, Kew"       |
    | file-metadata.csv   | date_last_modified                      | 2022-07-18T00:00:00                |
    | file-metadata.csv   | description                             |                                    |
    | file-metadata.csv   | language                                | English                            |
	| file-metadata.csv   | end_date                                |                                    |
	| file-metadata.csv   | file_name_translation                   |                                    |
	| file-metadata.csv   | original_filepath                       |                                    |
	| file-metadata.csv   | former_reference_department             |                                    |

When transform that bagit

Then the bagit leads to metadata.csv which has:

    | identifier                   | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt          | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv clientside_original_filepath])
    | file_name                    | file-a1.txt                                                      | # [file-metadata.csv FileName]
    | folder                       | file                                                             | # transformed [file-metadata.csv file_type]
    | date_last_modified           | 2022-07-18T00:00:00                                              | # [file-metadata.csv date_last_modified] (note that this will be different for a folder) 
    | description                  |                                                                  | # empty
	| end_date                     |                                                                  | # empty 	
    | checksum                     | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright             | Crown Copyright                                                  | # [file-metadata.csv rights_copyright]
    | legal_status                 | Public Record(s)                                                 | # transformed [file-metadata.csv legal_status]
    | held_by                      | "The National Archives, Kew"                                     | # transformed [file-metadata.csv held_by] 
    | language                     | English                                                          | # [file-metadata.csv language]
	| original_identifier          |                                                                  | # empty
	| file_name_translation        |                                                                  | # empty
    | TDR_consignment_ref          | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]
	| former_reference_department  |                                                                  | # empty

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content (note that we add a final "/" if a folder)
- remove capital of bagit field FileType to make dri field folder
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value
- other columns are included but are empty unless data exists in their corresponding bagit column.

## The metadata.csv row for a file with additional metadata

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime             | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |

    | manifest-sha256.txt | first column                            | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                           | data/content/folder-a/file-a1.txt  |

    | file-metadata.csv   | file_name                               | file-a1.txt                        |
	| file-metadata.csv   | file_type                               | File                               |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | rights_copyright                        | Crown Copyright                    |
    | file-metadata.csv   | legal_status                            | Public Record(s)                   |
    | file-metadata.csv   | held_by                                 | "The National Archives, Kew"       |
    | file-metadata.csv   | date_last_modified                      | 2022-07-18T00:00:00                |
    | file-metadata.csv   | description                             | Test description                   |
    | file-metadata.csv   | language                                | English                            |
	| file-metadata.csv   | end_date                                | 2022-07-18T00:00:00                |
	| file-metadata.csv   | file_name_translation                   | Translated title                   |
	| file-metadata.csv   | original_filepath                       |                                    |
	| file-metadata.csv   | former_reference_department             | former/1                           |

When transform that bagit

Then the bagit leads to metadata.csv which has:

    | identifier                   | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt          | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv clientside_original_filepath])
    | file_name                    | file-a1.txt                                                      | # [file-metadata.csv file_name]
    | folder                       | file                                                             | # transformed [file-metadata.csv file_type]
    | date_last_modified           | 2022-07-18T00:00:00                                              | # [file-metadata.csv date_last_modified] (note that this will be different for a folder) 
    | description                  | Test description                                                 | # [file-metadata.csv description]
	| end_date                     | 2022-07-18T00:00:00                                              | # [file-metadata.csv end_date]	
    | checksum                     | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright             | Crown Copyright                                                  | # [file-metadata.csv rights_copyright]
    | legal_status                 | Public Record(s)                                                 | # transformed [file-metadata.csv legal_status]
    | held_by                      | "The National Archives, Kew"                                     | # transformed [file-metadata.csv held_by] 
    | language                     | English                                                          | # [file-metadata.csv language]
	| original_identifier          |                                                                  | # empty
	| file_name_translation        | Translated title                                                 | # [file-metadata.csv file_name_translation]
    | TDR_consignment_ref          | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]
	| former_reference_department  | former/1                                                         | # [file-metadata.csv former_reference_department]

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content (note that we add a final "/" if a folder)
- remove capital of bagit field FileType to make dri field folder
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value
- other columns are included but are empty unless data exists in their corresponding bagit column.

## The metadata.csv row for a file with welsh and english language and a comma in filename and description

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime             | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |

    | manifest-sha256.txt | first column                            | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                           | data/content/folder-a/file-a1,.txt |

    | file-metadata.csv   | file_name                               | "file-a1-,.txt"                     |
	| file-metadata.csv   | file_type                               | File                                |
    | file-metadata.csv   | clientside_original_filepath            | "data/content/folder-a/file-a1,.txt"|
    | file-metadata.csv   | rights_copyright                        | Crown Copyright                     |
    | file-metadata.csv   | legal_status                            | Public Record(s)                    |
    | file-metadata.csv   | held_by                                 | "The National Archives, Kew"        |
    | file-metadata.csv   | date_last_modified                      | 2022-07-18T00:00:00                 |
    | file-metadata.csv   | description                             | "Test,description"                  |
    | file-metadata.csv   | language                                | Welsh|English                       |
	| file-metadata.csv   | end_date                                | 2022-07-18T00:00:00                 |
	| file-metadata.csv   | file_name_translation                   |                                     |
	| file-metadata.csv   | original_filepath                       |                                     |
	| file-metadata.csv   | former_reference_department             |                                     |

When transform that bagit

Then the bagit leads to metadata.csv which has:

    | identifier                   | "file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt"        | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv clientside_original_filepath])
    | file_name                    | "file-a1,.txt"                                                   | # [file-metadata.csv file_name]
    | folder                       | file                                                             | # transformed [file-metadata.csv file_type]
    | date_last_modified           | 2022-07-18T00:00:00                                              | # [file-metadata.csv date_last_modified] (note that this will be different for a folder) 
    | description                  | "Test,description"                                               | # transformed [file-metadata.csv description]
	| end_date                     | 2022-07-18T00:00:00                                              | # [file-metadata.csv end_date]	
    | checksum                     | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright             | Crown Copyright                                                  | # [file-metadata.csv rights_copyright]
    | legal_status                 | Public Record(s)                                                 | # transformed [file-metadata.csv legal_status]
    | held_by                      | "The National Archives, Kew"                                     | # transformed [file-metadata.csv held_by] 
    | language                     | English and Welsh                                                | # transformed [file-metadata.csv language]
	| original_identifier          |                                                                  | # empty
	| file_name_translation        |                                                                  | # [file-metadata.csv file_name_translation]
    | TDR_consignment_ref          | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]
	| former_reference_department  |                                                                  | # [file-metadata.csv former_reference_department]
	
and closure.csv with the same identifier column

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content (note that we add a final "/" if a folder), identifier will be enclosed in "" if a comma is present
- remove capital of bagit field FileType to make dri field folder
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value
- description will be enclosed in "" if a comma is present
- file_name will be enclosed in "" if a comma is present
- language if it contains english and welsh will be converted from 'English|Welsh' to 'English and Welsh', this behaviour may be different when TDR allows more languages

## The metadata.csv row for a redacted file

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime             | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |

    | manifest-sha256.txt | first column                            | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                           | data/content/folder-a/closed_file_R.pdf |

    | file-metadata.csv   | file_name                               | closed_file_R.pdf                        |
	| file-metadata.csv   | file_type                               | File                                     |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a/closed_file_R.pdf  |
    | file-metadata.csv   | rights_copyright                        | Crown Copyright                          |
    | file-metadata.csv   | legal_status                            | Public Record(s)                         |
    | file-metadata.csv   | held_by                                 | "The National Archives, Kew"             |
    | file-metadata.csv   | date_last_modified                      | 2022-07-18T00:00:00                      |
    | file-metadata.csv   | description                             |                                          |
    | file-metadata.csv   | language                                | English                                  |
	| file-metadata.csv   | end_date                                |                                          |
	| file-metadata.csv   | file_name_translation                   |                                          |
	| file-metadata.csv   | original_filepath                       | data/content/folder-a/closed_file.pdf    |
	| file-metadata.csv   | former_reference_department             |                                          |

When transform that bagit

Then the bagit leads to metadata.csv which has:

    | identifier                   | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/closed_file_R.pdf    | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv clientside_original_filepath])
    | file_name                    | closed_file_R.pdf                                                | # [file-metadata.csv file_name]
    | folder                       | file                                                             | # transformed [file-metadata.csv file_type]
    | date_last_modified           | 2022-07-18T00:00:00                                              | # [file-metadata.csv date_last_modified] (note that this will be different for a folder) 
    | description                  |                                                                  | # empty
	| end_date                     |                                                                  | # [file-metadata.csv end_date]	
    | checksum                     | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright             | Crown Copyright                                                  | # [file-metadata.csv rights_copyright]
    | legal_status                 | Public Record(s)                                                 | # transformed [file-metadata.csv legal_status]
    | held_by                      | "The National Archives, Kew"                                     | # transformed [file-metadata.csv held_by] 
    | language                     | English                                                          | # transformed [file-metadata.csv language]
	| original_identifier          | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/closed_file.pdf      | # (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv original_filepath])
	| file_name_translation        |                                                                  | # [file-metadata.csv file_name_translation]
    | TDR_consignment_ref          | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]
	| former_reference_department  |                                                                  | # [file-metadata.csv former_reference_department]
	
and closure.csv with the same identifier column

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content (note that we add a final "/" if a folder), identifier will be enclosed in "" if a comma is present
- remove capital of bagit field FileType to make dri field folder
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value
- original_identifier is adjusted the same as identifier apart from that it is populated by bagit value of original_filepath 
- if original_filpath includes a ',' then original_identifier should be enclosed with ""

## The metadata.csv row for a folder

Given a bagit with files that include:

    | File                | Field                                   | Value                              |
    | bag-info.txt        | Consignment-Series                      | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier              | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime             | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder    | false                              |
	
	# manifest-sha256.txt does not have an entry for a folder

    | manifest-sha256.txt | first column                            | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                           | data/content/folder-a              |

    | file-metadata.csv   | file_name                               | folder-a                           |
	| file-metadata.csv   | file_type                               | Folder                             |
    | file-metadata.csv   | clientside_original_filepath            | data/content/folder-a              |
    | file-metadata.csv   | rights_copyright                        | Crown Copyright                    |
    | file-metadata.csv   | legal_status                            | Public Record(s)                   |
    | file-metadata.csv   | held_by                                 | "The National Archives, Kew"       |
    | file-metadata.csv   | date_last_modified                      |                                    |
    | file-metadata.csv   | description                             |                                    |
    | file-metadata.csv   | language                                | English                            |
	| file-metadata.csv   | end_date                                | 2022-07-18T00:00:00                |
	| file-metadata.csv   | file_name_translation                   |                                    |
	| file-metadata.csv   | original_filepath                       |                                    |
	| file-metadata.csv   | former_reference_department             |                                    |

When transform that bagit

Then the bagit leads to metadata.csv which has:

    | identifier                   | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/                     | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv clientside_original_filepath])
    | file_name                    | folder-a                                                         | # [file-metadata.csv file_name]
    | folder                       | file                                                             | # transformed [file-metadata.csv file_type]
    | date_last_modified           | 2022-07-18T00:00:00                                              | # transformed [bag-info.txt Consignment-Export-Datetime] (note that this is a different source from file row)
    | description                  |                                                                  | # empty
	| end_date                     | 0                                                                | # empty 	
    | checksum                     | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright             | Crown Copyright                                                  | # [file-metadata.csv rights_copyright]
    | legal_status                 | Public Record(s)                                                 | # transformed [file-metadata.csv legal_status]
    | held_by                      | "The National Archives, Kew"                                     | # transformed [file-metadata.csv held_by] 
    | language                     | English                                                          | # [file-metadata.csv language]
	| original_identifier          |                                                                  | # empty
	| file_name_translation        |                                                                  | # empty
    | TDR_consignment_ref          | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]
	| former_reference_department  |                                                                  | # empty


Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" as a folder
- remove capital of bagit field FileType to make dri field folder
- the final "Z" is removed from file-metadata.csv's LastModified to provide dri field date_last_modified
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value
- other columns are included but are empty unless data exists in their corresponding bagit column.


## Consignment-Include-Top-Level-Folder

Given a bagit with files that include:

    | File                | Field                                | Value                              |
    | bag-info.txt        | Consignment-Series                   | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier           | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime          | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder | true                               |  

    | manifest-sha256.txt | first column                   | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                  | data/keepfolder/folder-a/file-a1.txt  |

    | file-metadata.csv   | Filepath                       | data/keepfolder/folder-a/file-a1.txt  |
	
When transform that bagit
	
Then the bagit leads to metadata.csv and closure.csv which has an identifier column as below:

    | identifier          | file:/TSTA1Y22TBAA1/TSTA_1/content/keepfolder/folder-a/file-a1.txt        | # same in closure.csv and metadata.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath])

Field Transforms: 
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + 'content' + the bagit identifier from after 'data' + a final "/" as a folder
- The payload files in bagit directory `data` are moved to SIP directory `batch/series/content`

Given a bagit with files that include:

    | File                | Field                                | Value                              |
    | bag-info.txt        | Consignment-Series                   | TSTA 1                             |
    | bag-info.txt        | Internal-Sender-Identifier           | TDR-2022-AA1                       | 
    | bag-info.txt        | Consignment-Export-Datetime          | 2022-07-18T12:45:45Z               | # used for folder dlm
	| bag-info.txt        | Consignment-Include-Top-Level-Folder | false                              |  

    | manifest-sha256.txt | first column                   | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                  | data/removefolder/folder-a/file-a1.txt  |

    | file-metadata.csv   | Filepath                       | data/removefolder/folder-a/file-a1.txt  |
	
When transform that bagit
	
Then the bagit leads to metadata.csv and closure.csv which has:

    | identifier          | file:/TSTA1Y22TBAA1/TSTA_1/content/folder-a/file-a1.txt        | # same in closure.csv and metadata.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath])

Field Transforms: 
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from after 'data' with the first folder renamed to 'content' + a final "/" as a folder
- The payload files in bagit directory `data` are moved to SIP directory `batch/series/content`




