tags:: azure, qpi
created:: [[Aug 10th, 2021]]

- https://docs.microsoft.com/en-us/rest/api/datafactory/v2
	- # Azure Data Factory API
	  collapsed:: true
	  
	  > With the Azure Data factory api I can capture run times and statuses to grafana.
	  I didn't find any capabilities for monitoring inside a job
	  
	  Can raise alerts on failures
	  
	   could build url into the notification to open the resource
		- kinds of built in I could reproduce
		  collapsed:: true
		  
		  ![Image.png](https://res.craft.do/user/full/1a24b3d7-de0a-7d28-2205-b3c808be324d/doc/4221C2DD-DCF8-423A-979B-02757D2042CE/2BDC758E-DB88-4988-93DB-5C0C3B4AC070_2/Image.png)
		  
		  ![Image.png](https://res.craft.do/user/full/1a24b3d7-de0a-7d28-2205-b3c808be324d/doc/4221C2DD-DCF8-423A-979B-02757D2042CE/B13725E3-C2B1-4066-BFB9-1BC7ACD198C2_2/Image.png)
		  
		  [Monitor data factories using Azure Monitor - Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/monitor-using-azure-monitor)
		  
		  ```other
		  ADF Runs - 1) Pipeline Runs by Data Factory
		  ADF Runs - 2) Activity Runs by Data Factory
		  ADF Runs - 3) Trigger Runs by Data Factory
		  ADF Errors - 1) Top 10 Pipeline Errors by Data Factory
		  ADF Errors - 2) Top 10 Activity Runs by Data Factory
		  ADF Errors - 3) Top 10 Trigger Errors by Data Factory
		  ADF Statistics - 1) Activity Runs by Type
		  ADF Statistics - 2) Trigger Runs by Type
		  ADF Statistics - 3) Max Pipeline Runs Duration
		  ```
		- Metrics list:
		  collapsed:: true
		  [Monitor data factories using Azure Monitor - Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/monitor-using-azure-monitor#data-factory-metrics)
		  
		  ![Image.png](https://res.craft.do/user/full/1a24b3d7-de0a-7d28-2205-b3c808be324d/doc/4221C2DD-DCF8-423A-979B-02757D2042CE/0CF8E417-D4B4-4645-B520-315E6868D108_2/Image.png)
- Factories - List By Resource Group
  collapsed:: true
	- Factories - List By Resource Group
	  collapsed:: true
		- [GET](https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories?api-version=2018-06-01)
			-
			  ```javascript
			  {
			    "value": [
			      {
			        "name": "exampleFactoryName-linked",
			        "identity": {
			          "type": "SystemAssigned",
			          "principalId": "10743799-44d2-42fe-8c4d-5bc5c51c0684",
			          "tenantId": "12345678-1234-1234-1234-123456789abc"
			        },
			        "id": "/subscriptions/12345678-1234-1234-1234-12345678abc/resourceGroups/exampleResourceGroup/providers/Microsoft.DataFactory/factories/exampleFactoryName-linked",
			        "type": "Microsoft.DataFactory/factories",
			        "properties": {
			          "provisioningState": "Succeeded",
			          "createTime": "2018-06-15T08:56:07.1828318Z",
			          "version": "2017-09-01-preview"
			        },
			        "eTag": "\"00008a02-0000-0000-0000-5b237f270000\"",
			        "location": "East US",
			        "tags": {}
			      },
			      {
			        "name": "FactoryToUpgrade",
			        "id": "/subscriptions/12345678-1234-1234-1234-12345678abc/resourceGroups/exampleResourceGroup/providers/Microsoft.DataFactory/factories/factorytoupgrade",
			        "type": "Microsoft.DataFactory/factories",
			        "properties": {
			          "provisioningState": "Succeeded",
			          "createTime": "2018-06-19T05:35:35.7133828Z",
			          "version": "2018-06-01"
			        },
			        "eTag": "\"00003d04-0000-0000-0000-5b28962f0000\"",
			        "location": "East US",
			        "tags": {}
			      },
			      {
			        "name": "exampleFactoryName",
			        "id": "/subscriptions/12345678-1234-1234-1234-12345678abc/resourceGroups/exampleResourceGroup/providers/Microsoft.DataFactory/factories/exampleFactoryName",
			        "type": "Microsoft.DataFactory/factories",
			        "properties": {
			          "provisioningState": "Succeeded",
			          "createTime": "2018-06-19T05:41:50.0041314Z",
			          "version": "2018-06-01",
			          "repoConfiguration": {
			            "type": "FactoryVSTSConfiguration",
			            "projectName": "project",
			            "tenantId": "",
			            "accountName": "ADF",
			            "repositoryName": "repo",
			            "collaborationBranch": "master",
			            "rootFolder": "/",
			            "lastCommitId": ""
			          }
			        },
			        "eTag": "\"00004004-0000-0000-0000-5b28979e0000\"",
			        "location": "East US",
			        "tags": {
			          "exampleTag": "exampleValue"
			        }
			      }
			    ]
			  }
			  ```
	- Call
		- `GET` [`https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories?api-version=2018-06-01`](https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DataFactory/factories?api-version=2018-06-01)``
	- URI Parameters
	  
	  ![Image.png](https://res.craft.do/user/full/1a24b3d7-de0a-7d28-2205-b3c808be324d/doc/4221C2DD-DCF8-423A-979B-02757D2042CE/9F96DB84-5599-4729-A822-B838726EC42F_2/Image.png)
	- Response
	  
	  ```javascript
	  {
	    "value": [
	      {
	   "name": "exampleFactoryName-linked",
	   "identity": {
	     "type": "SystemAssigned",
	     "principalId": "10743799-44d2-42fe-8c4d-5bc5c51c0684",
	     "tenantId": "12345678-1234-1234-1234-123456789abc"
	   },
	   "id": "/subscriptions/12345678-1234-1234-1234-12345678abc/resourceGroups/exampleResourceGroup/providers/Microsoft.DataFactory/factories/exampleFactoryName-linked",
	   "type": "Microsoft.DataFactory/factories",
	   "properties": {
	     "provisioningState": "Succeeded",
	     "createTime": "2018-06-15T08:56:07.1828318Z",
	     "version": "2017-09-01-preview"
	   },
	   "eTag": "\"00008a02-0000-0000-0000-5b237f270000\"",
	   "location": "East US",
	   "tags": {}
	      },
	      {
	   "name": "FactoryToUpgrade",
	   "id": "/subscriptions/12345678-1234-1234-1234-12345678abc/resourceGroups/exampleResourceGroup/providers/Microsoft.DataFactory/factories/factorytoupgrade",
	   "type": "Microsoft.DataFactory/factories",
	   "properties": {
	     "provisioningState": "Succeeded",
	     "createTime": "2018-06-19T05:35:35.7133828Z",
	     "version": "2018-06-01"
	   },
	   "eTag": "\"00003d04-0000-0000-0000-5b28962f0000\"",
	   "location": "East US",
	   "tags": {}
	      },
	      {
	   "name": "exampleFactoryName",
	   "id": "/subscriptions/12345678-1234-1234-1234-12345678abc/resourceGroups/exampleResourceGroup/providers/Microsoft.DataFactory/factories/exampleFactoryName",
	   "type": "Microsoft.DataFactory/factories",
	   "properties": {
	     "provisioningState": "Succeeded",
	     "createTime": "2018-06-19T05:41:50.0041314Z",
	     "version": "2018-06-01",
	     "repoConfiguration": {
	       "type": "FactoryVSTSConfiguration",
	       "projectName": "project",
	       "tenantId": "",
	       "accountName": "ADF",
	       "repositoryName": "repo",
	       "collaborationBranch": "master",
	       "rootFolder": "/",
	       "lastCommitId": ""
	     }
	   },
	   "eTag": "\"00004004-0000-0000-0000-5b28979e0000\"",
	   "location": "East US",
	   "tags": {
	     "exampleTag": "exampleValue"
	   }
	      }
	    ]
	  }
	  ```
- [TriggerRuns](https://docs.microsoft.com/en-us/rest/api/datafactory/trigger-runs/query-by-factory)
  collapsed:: true
  ```other
  {
  "value": [
    {
      "triggerName": "exampleTrigger",
      "triggerRunId": "08586724970898148904457116912CU27",
      "triggerType": "ScheduleTrigger",
      "triggerRunTimestamp": "2018-06-16T00:43:15.660141Z",
      "status": "Succeeded",
      "message": "",
      "properties": {
        "TriggerTime": "6/16/2018 12:43:15 AM",
        "ScheduleTime": "6/16/2018 12:43:14 AM"
      },
      "triggeredPipelines": {
        "examplePipeline": "9f3ce8b3-37d7-43eb-96ac-a656c0476283"
      }
    }
  ]
  }
  ```
	- Status
	  
	   Inprogress
	  
	   Failed
	  
	   Succeeded
- [PipeLineRuns](https://docs.microsoft.com/en-us/rest/api/datafactory/pipeline-runs/get)
  collapsed:: true
  ```json
  {
  "runId": "2f7fdb90-5df1-4b8e-ac2f-064cfa58202b",
  "pipelineName": "examplePipeline",
  "parameters": {
    "OutputBlobNameList": "[\"exampleoutput.csv\"]"
  },
  "invokedBy": {
    "id": "80a01654a9d34ad18b3fcac5d5d76b67",
    "name": "Manual"
  },
  "runStart": "2018-06-16T00:37:44.6257014Z",
  "runEnd": "2018-06-16T00:38:12.7314495Z",
  "durationInMs": 28105,
  "status": "Succeeded",
  "message": "",
  "lastUpdated": "2018-06-16T00:38:12.7314495Z",
  "annotations": []
  }
  ```