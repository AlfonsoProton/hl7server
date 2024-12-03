# hl7server
HL7 Asyncronous Server built in Python
### The HL7 server is a fast and reliable service built with asynchronous Python and designed to run on Debian 12.

Key features:

- ***Multiple channels**: Handles several channels at once, each processing different types of HL7 messages.*
- ***High performance**: Processes multiple messages per second, depending on CPU and storage speed.*
- ***Lightweight**: Efficient in memory usage, even under heavy use.*

Currently supported message types:

- ADT^A01 → Will generate patient credentials
- ADT^A08 → Will generate patient credentials
- ADT^A34 → Will update patient data (work in progress)
- ORM^O01 → Will generate patient credentials
- ORU^R01 → Will extract attachments, add metadata and deliver to a folder
- MDM^T02 → Will extract attachments, add metadata and deliver to a folder

### Service → Global service configuration

- **debug_values**: `[true|false]` → provides HL7 field debugging globally.
    - Screenshot sample
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/35c8da53-482a-4a42-8f74-f3c5c7073806/931a8f85-d87e-409b-94ca-7878e45c4193/image.png)
        
- **debug_mode**:  `[true|false]` → provides details debug information of every method called.
- **config_version**:  `String` → current version of configuration file.
- **max_tasks**:  `Int` → maximum number of simultaneous tasks (processed messages)
- **store_solr**: `[true|false]` → if metadata added to the SolR core hl7.
- **create_credentials**: `[true|false]` → service will be creating credentials calling dfapi.
- **solr_servers**: `List of Dicts` → list of Solr servers used. Currently only one server is supported.
    - server_version: `String` → Supported Solr version. Currently only version 7.7.1 is supported.
    - name: `String` → FQDN (can be IP address). Default is localhost.
    - port: `Int` →  If null default value will be 8983.
    - protocol: `[String|null]` → Default value is https.
    - user: `[String|null]`  → If null default value will be istmia
    - password: `[String|null]` →  If null it will take the environment variable value of SOLR_TOKEN
    - certificate:  `[true|false]`   → If false it will bypass the need of a valid https certificate.
    - active: `[true|false]`  → If false this server is disabled. If field not found it will be enabled.
    - active_cores:  `List of Strings` → Cores that will be initialized and validated at service start
- **channels**: `List of Dicts | !include` → List of channels. Explained in section [Channel](section_channels). Can use the keyword include to reference external files.

### Channel → Chanel specific configuration

- **debug_message**: `[true|false]` → provides message debugging per channel.
    - Screenshot sample
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/35c8da53-482a-4a42-8f74-f3c5c7073806/8888d13c-ac99-4050-be5c-f9a36f471be3/image.png)
        
- **name**: `String`. Unique channel name.
- **port**:  Unique channel port.  Value must be an integer in the range: `1024–49151` and open in the firewall. For testing we use from 20000 to 29000.
- **stream_limit**: `Integer`. Message size. The larger the value, the more memory is used per message. Usually ADT is fine with 512. If attachments are included sizes can grow to 4194304 or even 10094304.
- **enabled**: `[true|false]` → if the channel is enabled. If the field does not exist the value is true.
- **debug_metadata**: `[true|false]` → Will display the metadata variable.
- 
- **patient_id_field:**  `String`. A valid HL7 field. Example: PID.1.3.1. Needed if we want to have the patient ID in the filename. Must be archive_list = false.
- **accession_number_field:**  `String`. A valid HL7 field. Example: OBX.1.3.1. Needed if we want to have the accession number in the filename. Must be archive_list = false.
- **attachment_field:**  `String`. A valid HL7 field. Example: OBX.1.5
- **attachment_format**: `[’pdf’|’text’]`. In what format the attach
- **archive_first**: `[true|false]` → If the message is archived before being processed. No metadata will be included in the filename if set to true.
- **encoding**: `String`. A valid encoding format. Default if field not existing is UTF-8.
- **add_metadata_to_attachment**: `[true|false]` → If metadata will be added to the attachment.
- **visible_metadata**: `[true|false]` → if True, will show the metadata in red.  Otherwise it will be transparent.
- **local_archive**: `Dict` → If the the HL7 message and or the message attachment will be archived locally.
    - enabled: `[true|false]` → If enabled. Default is true even if the enabled field does not exist.
    - format: `String` → How files will be stored. Must be a valid strftime string. Default if field not indicated is `'%Y/%m/%d’`.
    - path:  `String` → root path where the messages will be archived. If path does not exist it will be created.
    - attachment:  `String`→ root path where the attachments will be archived. If path does not exist it will be created.
    - Examples
        
        ```yaml
        local_archive:
          enabled: true
          format: '%Y/%m/%d'
          path: /opt/archive/hl7server/oru
          attachment: /opt/archive/hl7server/oru_pdf
        ```
        
- **drop_folders**: `List of Dicts` → Locations where the extracted and metadata-modified attachment or credentials task will be dropped.
    
    Each location is a Dict and may contain the following fields:
    
    - enabled: `[true|false]` → If enabled. Default is true even if the enabled field does not exist.
    - remote: `[true|false]` → If remote. Default is false even if the remote field does not exist.
    - path:  `String` → root path where the attachments will be dropped. If path does not exist it will be created.
    - server: `String` → If remote is true, the server where the attachment will be dropped.
    - port: `Int` →  If remote is true, port to access the remote server via SSH.
    - user: `String` →  If remote is true, user to access the remote server via SSH.
    - Examples
        
        ```yaml
        drop_folders:
          localhost:
            enabled: true
            remote: false
            path: /opt/data/documents
          remote:
            enabled: true
            remote: true
            path: /opt/data/documents
            server: remote-server
            port: 22
            user: remote-user
        ```
        
- **mapping**: `Dict` → List of fields to be extracted from the HL7 message and the internal name associated to the value.
    
    Each fied can be a string or a list of strings. When a list of strings, values will be concatenated together. 
    
    - Examples
        
        ```yaml
          patient_id: PID.1.2.1
          accession_number: OBR.1.18.1
          patient_name:  [PID.1.5.1, PID.1.5.2]
          referring_doctor_id: PV1.1.8.1
          referring_doctor_name: [PV1.1.8.2, PV1.1.8.3]
        ```
        
- **mapping_ignore**: `List` → List of mapping fields to ignore if not found. An error message will not be generated and the workflow will continue.
- **db_metadata** `Dict of Dicts` → will insert values in the database indicated in db_insert field
    
    Each key in the dictionary corresponds to a table column to be inserted in the table indicated with [db_insert](db_insert). 
    
    They key name can be any of:
    
    - A [db_mapping](db_mapping) name, extracted from another table.
    - A metadata value extracted from [mapping](mapping) data.
    - An [automatic value](automatic_value) that is generated when the auto option is true.
    - **field_name**: `String` → The row name in the database. Must match exactly this name.
    - **field_type**: `String` → The type of field. A valid SQL field type. Valid values are: Integer, String(50), Text, DateTime
    - **autoincrement**:`[true|false]` → If this field automatically increments.
    - **random**:`[true|false]` → If this field automatically generates a random number.
    - **nullable**:`[true|false]` → If this field can have a null value.
    - **auto**:`[true|false]` → If this field can be completed automatically. Only supported for field_type DateTime
- **db_insert**: `Dict` → configuration to access the database where metadata will be inserted
    - **driver**: `String` → Driver type for SQL Alchemy. (Currently supported two types: MariaDB, ODBC Driver 17 for SQL Server)
    - **server**: `String` → FQDN (can be IP address). Default is localhost.
    - **port**: `Int` → Must be provided
    - **username**: `String` → Must be provided
    - **password**: `String` → Reference to an environment variable with a tokenized value
    - **database**: `String` → The database name
    - **table**: `String` → The table name
    - **mapping_table**: `String` → The table used for mapping values if db_mapping is configured
    - **create_table**:`[true|false]` → If the table is to be created if not existing
    - **debug_table**: `[true|false]` → If table debug information will be provided in the logfile
    - **replace_list**: `Dict of Lists` → fields in db_metadata to be replaced
    - Examples
        
        ```yaml
        db_insert:
          driver: MariaDB
          server: db-server-url
          port: 12345
          username: deskuser
          password: DB_TOKEN
          database: PACS_DB
          table: reports
          mapping_table: worklist
          create_table: true
          debug_table: true
          replace_list: { REPORT: ['<br>', '\r\n'] }
        ```
        
- **db_mapping**:  `Dict` → configuration to access the database where metadata will be inserted
    - **column**: `String →` The column to search in table [mapping_table](mapping_table)
    - **match**:`String →` The value to match from the [match_table) extracted from the message.
    - **return**:`String →` The column with the value to return. This value will be added to the key value as an internal variable.
    - Examples
        
        ```yaml
        # In this example, new_patient_id can be used as a metadata value in db_metadata, credentials_metadata, book_metadata or mobile -> fields
        db_mapping:
          new_patient_id: { column: WL_ACCNUMBER, match: accession_number, return: WL_PATIENTID }
        ```
        
- **custom_regex**: `Dict` → List of regular expresions to apply to the field after extraction from HL7 message.
    - Examples
        
        ```yaml
        custom_regex:
          patient_number_home: "[^0-9+]|(?<!^)\\+"
          patient_number_business: "[^0-9+]|(?<!^)\\+"
        ```
        
- **book_metadata**: `Dict`
- **resend**: `Dict` → Will resend the message to the indicated location
    - **port**: `Int` → Must be provided
    - **server**: `String` → FQDN (can be IP address).
    - **timeout**: `Int` → Timeout in seconds to wait for the message to be sent.
    - **enabled**: `[true|false]` → If resend is enabled.
    - **after_processing**:  `[true|false]` → Will resend the message after processing it locally. Otherwise it will resend once received. Default is false.
    - **filter**: `Dict` → Filter to apply for resending messages
        - **source_core**:`String` → Solr Core where to search for the value of match_source in match_field.
        - **match_source**:`String` → The HL7 field where to extract the value to search in match_field.
        - **match_field**:`String` → The field in Solr Core where to search the value of match_source.
        - **search_field**: `String` → The field in Solr Core where to search for the match values.
        - **match_values**: `List` → List of values to matched in search_field. If match is found message will be forwarded.
    - Example
        
        ```yaml
        resend:
          enabled: true
          server: remote-server
          port: 20002
          timeout: 5
          after_processing: true
          filter:
            source_core: archive
            match_source: OBX.1.3.2
            match_field: StudyInstanceUID
            search_field: InstitutionName
            match_values: ['CENTER_NAME', 'MEDICALCENTER_NAME', 'FACILITY_NAME']
        
        ```
        
- **solr_segments**: `List` → List of segments to be archived in Solr. Value values are: MSH, PID, OBX, OBR, ZDS, TXA, ORC, PV1, PV2, EVN, CTI, NK1, AL1, DG1, PR1, ROL, GT1, IN1

