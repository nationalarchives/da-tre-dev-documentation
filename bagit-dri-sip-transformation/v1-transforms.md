# Overall structure of SIP

- The payload files in bagit directory `data` are moved to SIP directory `batch/series/content` (there will, later, be code to deal with situation where `data` dir does not start with `content`)
- `closure.csv` and `metadata.csv` files are added to the `batch/series/` directory of the SIP following spec below (and they have a checksum file added alongside them, see sample data for example of files & their locations in SIP)
- `closure.csvs` and `metadata.csvs` files are copied to same dir (currently from hardcoded file)

# Fields from bagit used to make the SIP

| Bagit File          | Field                          | Example Value                      | Batch ID | closure.csv | metadata.csv |
| ---                 | ---                            | ---                                | ---      | ---         | ---          |
| file-metadata.csv   | Filepath                       | data/content/folder-a/file-a1.txt  |          |    X        |       X      |
|                     | FileName                       | file-a1.txt                        |          |             |       X      |
|                     | FileType                       | File                               |          |    X        |       X      |
|                     | Filesize                       | 70263                              |          |             |              |
|                     | RightsCopyright                | Crown Copyright                    |          |             |       X      |
|                     | LegalStatus                    | Public Record                      |          |             |       X      |
|                     | HeldBy                         | TNA                                |          |             |       X      |
|                     | Language                       | English                            |          |             |       X      |
|                     | FoiExemptionCode               | open                               |          |    X        |              |
|                     | LastModified                   | 2022-07-18T00:00:00                |          |             |       X      |
| --- |
| manifest-sha256.txt | first column                   | 4ef13 ... a34b                     |          |             |       X      |
|                     | second column                  | data/content/folder-a/file-a1.txt  |          |             |              |
| --- |
| bag-info.txt        | Consignment-Type               | standard                           |          |             |              |
|                     | Bag-Creator                    | TDRExportv0.0.107                  |          |             |              |
|                     | Consignment-Start-Datetime     | 2022-07-18T12:39:41Z               |          |             |              |
|                     | Consignment-Series             | MOCKA 101                          |    X     |    X        |       X      |
|                     | Source-Organization            | MOCKA                              |          |             |              |
|                     | Contact-Name                   | DA Example                         |          |             |              |
|                     | Internal-Sender-Identifier     | TDR-2022-AA1                       |    X     |    X        |       X      |
|                     | Consignment-Completed-Datetime | 2022-07-18T12:44:52Z               |          |             |              |
|                     | Consignment-Export-Datetime    | 2022-07-18T12:45:45Z               |          |             |       X      |
|                     | Contact-Email                  | da.example@nationalarchives.gov.uk |          |             |              |
|                     | Payload-Oxum                   | 3893083.18                         |          |             |              |
|                     | Bagging-Date                   | 2022-07-18                         |          |             |              |

# Constructing the SIP

## The Batch / Series are make the path `batch/series`:

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |

When transform that bagit

Then dri batch is named `MOCKA101Y22TBAA1`

Transformation in this example is:
+ `MOCKA101` (Consignment-Series with the space removed)
+ `Y`
+ `22` (2 digits from year Internal-Sender-Identifier)
+ `T`
+ `B`
+ `AA1` (final "base 25" part of Internal-Sender-Identifier)

This batch name is used with the series name to construct the base directories in the sip: `batch/series` and as part
of the `identifier` fields of `closure.csv` and `metadata.csv`. (The series name part of the path is simply the value
from `bag-info.txt`'s `Consignment-Series` with the space removed)

# Constructing closure.csv / metadata.csv rows for a file

## The closure.csv row for a file

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |
    | file-metadata.csv   | Filepath                       | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | FileType                       | File                               |
    | file-metadata.csv   | FoiExemptionCode               | open                               |

When transform that bagit

Then the bagit leads to closure.csv which has:

    | identifier             | file:/MOCKA101Y22TBAA1/MOCKA_101/content/folder-a/file-a1.txt    | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath]
    | folder                 | file                                                             | # transformed [file-metadata.csv FileType]
    | closure_start_date     |                                                                  | # always empty
    | closure_period         | 0                                                                | # always 0
    | foi_exemption_code     | open                                                             | # [file-metadata.csv FoiExemptionCode]
    | foi_exemption_asserted |                                                                  | # always empty
    | title_public           | TRUE                                                             | # always TRUE
    | title_alternate        |                                                                  | # always empty
    | closure_type           | open_on_transfer                                                 | # always open-on-transfer

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" if a folder
- remove capital of bagit field FileType to make dri field folder

## The closure.csv row for a folder

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |
    | file-metadata.csv   | Filepath                       | data/content/folder-a              |
    | file-metadata.csv   | FileType                       | Folder                             |
    | file-metadata.csv   | FoiExemptionCode               | open                               |

When transform that bagit

Then the bagit leads to closure.csv which has:

    | identifier             | file:/MOCKA101Y22TBAA1/MOCKA_101/content/folder-a/               | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath]
    | folder                 | folder                                                           | # transformed [file-metadata.csv FileType]
    | closure_start_date     |                                                                  | # always empty
    | closure_period         | 0                                                                | # always 0
    | foi_exemption_code     | open                                                             | # [file-metadata.csv FoiExemptionCode]
    | foi_exemption_asserted |                                                                  | # always empty
    | title_public           | TRUE                                                             | # always TRUE
    | title_alternate        |                                                                  | # always empty
    | closure_type           | open_on_transfer                                                 | # always open-on-transfer

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" as a folder
- remove capital of bagit field FileType to make dri field folder

## The metadata.csv row for a file

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |
    | bag-info.txt        | Consignment-Export-Datetime    | 2022-07-18T12:45:45Z               | # used for folder dlm

    | manifest-sha256.txt | first column                   | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b |
    | manifest-sha256.txt | second column                  | data/content/folder-a/file-a1.txt  |

    | file-metadata.csv   | Filepath                       | data/content/folder-a/file-a1.txt  |
    | file-metadata.csv   | FileType                       | File                               |
    | file-metadata.csv   | LegalStatus                    | Public Record                      |
    | file-metadata.csv   | HeldBy                         | TNA                                |
    | file-metadata.csv   | FileName                       | file-a1.txt                        |
    | file-metadata.csv   | LastModified                   | 2022-07-18T00:00:00                |
    | file-metadata.csv   | RightsCopyright                | Crown Copyright                    |
    | file-metadata.csv   | Language                       | English                            |

When transform that bagit

And the bagit leads to metadata.csv which has:

    | identifier             | file:/MOCKA101Y22TBAA1/MOCKA_101/content/folder-a/file-a1.txt    | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath])
    | file_name              | file-a1.txt                                                      | # [file-metadata.csv FileName]
    | folder                 | file                                                             | # transformed [file-metadata.csv FileType]
    | date_last_modified     | 2022-07-18T00:00:00                                              | # [file-metadata.csv LastModified] (note that this will be different for a folder)
    | checksum               | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright       | Crown Copyright                                                  | # [file-metadata.csv RightsCopyright]
    | legal_status           | Public Record(s)                                                 | # transformed [file-metadata.csv LegalStatus]
    | held_by                | "The National Archives, Kew"                                     | # transformed [file-metadata.csv HeldBy]
    | language               | English                                                          | # [file-metadata.csv Language]
    | TDR_consignment_ref    | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content (note that we add a final "/" if a folder)
- remove capital of bagit field FileType to make dri field folder
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value

## The metadata.csv row for a folder

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |
    | bag-info.txt        | Consignment-Export-Datetime    | 2022-07-18T12:45:45Z               | # used for folder dlm

    # manifest-sha256.txt does not have an entry for a folder

    | file-metadata.csv   | Filepath                       | data/content/folder-a              |
    | file-metadata.csv   | FileType                       | Folder                             |
    | file-metadata.csv   | FileName                       | folder-a                           |
    # these are the same as we see for a file row
    | file-metadata.csv   | LegalStatus                    | Public Record                      |
    | file-metadata.csv   | HeldBy                         | TNA                                |
    | file-metadata.csv   | RightsCopyright                | Crown Copyright                    |
    | file-metadata.csv   | Language                       | English                            |

When transform that bagit

And the bagit leads to metadata.csv which has:

    | identifier             | file:/MOCKA101Y22TBAA1/MOCKA_101/content/folder-a/               | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath])
    | file_name              | folder-a                                                         | # [file-metadata.csv FileName]
    | folder                 | folder                                                           | # transformed [file-metadata.csv FileType]
    | date_last_modified     | 2022-07-18T00:00:00                                              | # transformed[bag-info.txt Consignment-Export-Datetime] (not that this is different source from a file row)
    | checksum               |                                                                  | # always blank for a folder
    | rights_copyright       | Crown Copyright                                                  | # [file-metadata.csv RightsCopyright]
    | legal_status           | Public Record(s)                                                 | # transformed [file-metadata.csv LegalStatus]
    | held_by                | "The National Archives, Kew"                                     | # transformed [file-metadata.csv HeldBy]
    | language               | English                                                          | # [file-metadata.csv Language]
    | TDR_consignment_ref    | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]

Field Transforms:
- identifier is built as "file:/" + batch + "/" + Consignment-Series with space made an underscore + "/" + the bagit identifier from content + a final "/" as a folder
- remove capital of bagit field FileType to make dri field folder
- the final "Z" is removed from file-metadata.csv's LastModified to provide dri field date_last_modified
- if bagit value of field legal_status is Public Record append "(s)" to make Public Record(s) otherwise use bagit value
- if bagit value of held_by is TNA then change to "The National Archives, Kew" otherwise use bagit value
