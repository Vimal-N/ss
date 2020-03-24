# Upsert Process
To create a upsert job, we find a External ID field on the object. Salesforce uses this field to check for the existing field in the Object and this field also needs to be provided in the batch job creation request

------

## Different parts to the Bulk API 2.0 operation
- Create batch Job and get job ID
- Upload data to the Job
- Mark the data upload complete
- Get Job result
- Get Errors from the Job


# Create a Job

## Request Headers
```Json
Content-Type: application/json; charset=UTF-8
Accept: application/json
```
## Request End Point
`/services/data/vXX.X/jobs/ingest/`

Construct the request body to create a job
-----
```Json
{
    "object" : "Contact",
    "externalIdFieldName" : "name_of_external_id_field",
    "contentType" : "CSV",
    "operation" : "upsert"
}
```

## Close inspection of the request body

`object` - Provide the name of the object for which you want to upsert records

`externalIdFieldName`  - Name of the external field on the Object using which Salesforce for check for existing records [check for api name of the field inside Object Manager -> Object -> Fields & Relationship] 

`contentType` - Provide information on the structure of your data. '`CSV`' for comma separate file, '`JSON`' for the Json formatted data.

`operation` - What kind of operation do you want to perform on the Object. in this case we want to do  **`upsert`**

## Response from Salesforce
```JSON

{
  "id" : "7502M00000aFzAIQA0",
  "operation" : "upsert",
  "object" : "LObjectScore__c",
  "createdById" : "005410000020mxHAAQ",
  "createdDate" : "2020-03-21T20:03:25.000+0000",
  "systemModstamp" : "2020-03-21T20:03:25.000+0000",
  "state" : "Open",
  "externalIdFieldName" : "values__c",
  "concurrencyMode" : "Parallel",
  "contentType" : "CSV",
  "apiVersion" : 47.0,
  "contentUrl" : "services/data/v47.0/jobs/ingest/7502M00000aFzAIQA0/batches",
  "lineEnding" : "LF",
  "columnDelimiter" : "COMMA"
}

```
## Close inspection of the response body
- `id` - Job ID which is created for this request. 
    - This ID is needed to send 'PUT' request to upload data
    - Set the Job ready for Salesforce process
    - Get Job state during its execution
    - To fetch result of the bulk process
    - To fetch error in event of failure

- `state` - State of job. 
    - **Open** - New Job is created
    - **Aborted** - Job is terminated by user
    - **UploadComplete** - data upload is completed. this mark job is ready for salesforce bulk process
    - **Failed** - Job failed on the execution and it has Errors

## Sample of Incorrect Request and its response 
- Invalid `object` Field value or providing Object which does not exist in the org
    ### Resquest Body
    ```JSON
    {
    "object" : "",
    "externalIdFieldName" : "values__c",
    "contentType" : "CSV",
    "operation" : "upsert"
    }
    ```
    ### Response 

    ```JSON
    [
        {
        "errorCode" : "INVALIDJOB",
        "message" : "InvalidJob : Unable to find object: "
        }
    ]
    ```    

- Invalid `externalIdFieldName`
    ### Resquest Body
    ```JSON
    {
    "object" : "LObjectScore__c",
    "externalIdFieldName" : "",
    "contentType" : "CSV",
    "operation" : "upsert"
    }
    ```
    ### Response 

    ```JSON
    [
        {
        "errorCode" : "INVALIDJOB",
        "message" : "InvalidJob : Field name provided,  does not match an External ID for LObjectScore__c"
        }
    ]
    ```

- Invalid `operation` 
    ### Resquest Body
    ```JSON
    {
    "object" : "LObjectScore__c",
    "externalIdFieldName" : "values__c",
    "contentType" : "CSV",
    "operation" : ""
    }
    ```
    ### Response 

    ```JSON
    [ 
        {
        "errorCode" : "API_ERROR",
        "message" : "Invalid value: '' supplied for property: 'operation'"
        } 
    ]
    ```    
- Invalid `contentType` 
    ### Resquest Body
    ```JSON
    {
    "object" : "LObjectScore__c",
    "externalIdFieldName" : "values__c",
    "contentType" : "CSV",
    "operation" : ""
    }
    ```
    ### Response 

    ```JSON
    [ 
        {
        "errorCode" : "API_ERROR",
        "message" : "Invalid value: '' supplied for property: 'contentType'"
        } 
    ]
    ```    

# Upload data
**IMPORTANT:** 
Data upload for each job ID can be done only once. if multiple data upload request is sent then Bulk API will return `BULK_API_ERROR`

```JSON
[ {
  "errorCode" : "BULK_API_ERROR",
  "message" : "Found multipe contents for job: <7502M00000aFzAI>, please 'Close' / 'Abort' / 'Delete' the current Job then create a new Job and make sure you only do 'PUT' once on a given Job."
} ]
```
## Adding Data to the job
## Request Header
```Json
Content-Type: text/csv
Accept: application/json
```

## Request End Point
Protocol    : `PUT`  
URI         : `/services/data/vXX.X/jobs/ingest/JOB ID/batches/`

Bulk 1.0    : `https://instance_name.salesforce.com/services/async/48.0/job`

- **vXX.X** - is the version of the data api which is available to your org.
- **JOB ID** - JOB Id which is returned in response body of Create Job request

## Request Body


# Mark the Job ready for processing

## Request Header
```Json
Content-Type: application/json; charset=UTF-8
Accept: application/json
```

## Request End Point
Protocol    : `PATCH`  
URI         : `/services/data/vXX.X/jobs/ingest/JOB ID/`

- **vXX.X** - is the version of the data api which is available to your org.
- **JOB ID** - JOB Id which is returned in response body of Create Job request

## Request Body
```JSON
{
    "state" : "UploadComplete"
}
```
## Response Body
```JSON
{
  "id" : "7502M00000aFzAIQA0",
  "operation" : "upsert",
  "object" : "LObjectScore__c",
  "createdById" : "005410000020mxHAAQ",
  "createdDate" : "2020-03-21T20:03:25.000+0000",
  "systemModstamp" : "2020-03-21T20:03:25.000+0000",
  "state" : "UploadComplete",
  "externalIdFieldName" : "values__c",
  "concurrencyMode" : "Parallel",
  "contentType" : "CSV",
  "apiVersion" : 47.0
}
```
