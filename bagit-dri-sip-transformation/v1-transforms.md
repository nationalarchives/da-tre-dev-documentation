## These fields from the bagit are used to make the SIP  

| Bagit File          | Field                          | Example Value                      | Batch ID | Closure.csv | Metadata.csv | 
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

# The batch is built like this:

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |

When transform that bagit

Then dri batch is named MOCKA101Y22TBAA1



## The closure.csv file is built like this

Given a bagit with files that include:

    | File                | Field                          | Value                              |
    | bag-info.txt        | Consignment-Series             | MOCKA 101                          |
    | bag-info.txt        | Internal-Sender-Identifier     | TDR-2022-AA1                       |
    | file-metadata.csv   | Filepath                       | data/content/folder-a/file-a1.txt  | 
    | file-metadata.csv   | FileType                       | File                               | 
    | file-metadata.csv   | FoiExemptionCode               | open                               |

When transform that bagit

And closure.csv has:

    | identifier             | file:/MOCKA101Y22TBAA1/MOCKA_101/content/folder-a/file-a1.txt    | # batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath]
    | folder                 | file                                                             | # transformed [file-metadata.csv FileType] (lower case bagit value)
    | closure_start_date     |                                                                  | # always empty
    | closure_period         | 0                                                                | # always 0
    | foi_exemption_code     | open                                                             | # [file-metadata.csv FoiExemptionCode] 
    | foi_exemption_asserted |                                                                  | # always empty
    | title_public           | TRUE                                                             | # always TRUE
    | title_alternate        |                                                                  | # always empty
    | closure_type           | open_on_transfer                                                 | # always open-on-transfer

[^1]: remove capital of bagit field FileType to make dri field folder

## The metadata.csv file is built like this

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

And metadata.csv has:

    | identifier             | file:/MOCKA101Y22TBAA1/MOCKA_101/content/folder-a/file-a1.txt    | # same as closure.csv (batch + transformed [bag-info.txt Consignment-Series] + transformed [file-metadata.csv Filepath])
    | file_name              | file-a1.txt                                                      | # [file-metadata.csv FileName]
    | folder                 | file                                                             | # transformed [file-metadata.csv FileType] (lower case bagit value)
    | date_last_modified     | 2022-07-18T00:00:00                                              | # [file-metadata.csv LastModified]
    | checksum               | 4ef13f1d2350fe1e9f79a88ec063031f65da834e8afdd0512e230544cca0a34b | # [manifest-sha256.txt first column]
    | rights_copyright       | Crown Copyright                                                  | # [file-metadata.csv RightsCopyright]
    | legal_status           | Public Record(s)                                                 | # transformed [file-metadata.csv LegalStatus] (append "(s)" to bagit value)
    | held_by                | "The National Archives, Kew"                                     | # transformed [file-metadata.csv HeldBy] (value of TNA in bagit is switched to "The National Archives, Kew") 
    | language               | English                                                          | # [file-metadata.csv Language]
    | TDR_consignment_ref    | TDR-2022-AA1                                                     | # [bag-info.txt Internal-Sender-Identifier]
