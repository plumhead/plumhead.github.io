---
title: This is my title
layout: post
---
# Data Request Routing

Information for running contract and statement calculations is retrieved from a variety of places, GPMS, Powerhub and legacy database being the main sources. The system design has been structured so that, in general, specific data hosts provide subsets of the required data and it is the JobHosts responsibility to decide which data requests to dispatch to which host to build up a complete request which can subsequently be processed by the calculation engines.

The main hosts for the provision of data are the Dynamic, Market, Asset and Ascm hosts - some data is provided by multiple hosts (eg MEL) and the choice of which will depend on the study period `prompt` or `daily` applicable to the job being processed. 

*The data provisioning is a little disjointed in places with elements residing in hosts to which the type of data doesn't really belong - a good task would be to tidy this part up such that the ultimate routing is performed by the `datahost` itself rather than the `jobhost`*. 

##Routing Configuration

The routing configuration is held in a config file `routing-config.json` provided as part of the `JobHost` application. The structure of this file is as defined below: -

***Note - this file is incomplete and provided for an example of structure only***
```json
[
    {
        "Scenario": "Global",        
        "Routes": [
            { "target": "instructions", "desc": "Instructions", "source-uri": "{ASCM_HOST}instructions" },
            { "target": "reactiveShortfall", "desc": "Reactive Power Shortfall", "source-uri": "{ASCM_HOST}reactiveShortfall" },           
            { "target": "dashboard", "desc": "Daily Job Dashboard", "source-uri": "{ASCM_HOST}dashboard" },      
            { "target": "notifications", "desc": "Notifications", "source-uri": "{ASCM_HOST}notifications" },      
            { "target": "savestatement", "desc": "Save Statement", "source-uri": "{ASCM_HOST}savestatement" },           
            { "target": "metadata", "desc": "Metadata", "source-uri": "{ASCM_HOST}metadata" },            
            { "target": "statements", "desc": "Statements", "source-uri": "{ASCM_HOST}statements" },         
            { "target": "rawstatements", "desc": "Raw Statements", "source-uri": "{ASCM_HOST}rawstatements" },           
            { "target": "delstmt", "desc": "Delete Statement", "source-uri": "{ASCM_HOST}delstmt" },          
            { "target": "stmtview", "desc": "Statement View", "source-uri": "{ASCM_HOST}stmtview" },         
            { "target": "addstmt", "desc": "Add Statement", "source-uri": "{ASCM_HOST}addstmt" },           
            { "target": "stmtdrilldown", "desc": "Statement Drilldown", "source-uri": "{ASCM_HOST}stmtdrilldown" },           
            { "target": "uploadstmt", "desc": "Upload Statement", "source-uri": "{ASCM_HOST}uploadstmt" },         
            { "target": "frstate", "desc": "FR State", "source-uri": "{ASCM_HOST}frstate" },           
            { "target": "bmsustate", "desc": "BMSU State", "source-uri": "{ASCM_HOST}bmsustate" },            
            { "target": "rpstate", "desc": "RP State", "source-uri": "{ASCM_HOST}rpstate" },        
            { "target": "contract", "desc": "Contract", "source-uri": "{ASCM_HOST}contract" },           
            { "target": "stationunits", "desc": "Stations and Units", "source-uri": "{ASCM_HOST}stationunits" },            
            { "target": "targetFrequency", "desc": "Target Frequency", "source-uri": "{ASCM_HOST}targetfrequency" },           
            { "target": "speriods", "desc": "Settlement Periods", "source-uri": "{ASCM_HOST}speriods" },            
            { "target": "storcalendar", "desc": "STOR Calendar", "source-uri": "{ASCM_HOST}storcalendar" }.            
            { "target": "revenue", "desc": "Revenue", "source-uri": "{ASCM_HOST}revenue" },         
            { "target": "sourcerevenue", "desc": "Source Revenue", "source-uri": "{ASCM_HOST}sourcerevenue" },            
            { "target": "storavailability", "desc": "STOR Availability", "source-uri": "{ASCM_HOST}storavailability" },
            { "target": "stormonth", "desc": "STOR Month", "source-uri": "{ASCM_HOST}stormonth" },
            { "target": "storseason", "desc": "STOR Season", "source-uri": "{ASCM_HOST}storseason" },
            { "target": "storyear", "desc": "STOR Year", "source-uri": "{ASCM_HOST}storyear" },
            { "target": "bmsudetail", "desc": "BMSU Detail", "source-uri": "{ASCM_HOST}bmsudetail" },
            { "target": "remigrate", "desc": "Remigrate", "source-uri": "{ASCM_HOST}remigrate" },
            { "target": "redeclarations", "desc": "Redeclarations", "source-uri": "{ASCM_HOST}redeclarations" },
            { "target": "addredec", "desc": "Add Redeclaration", "source-uri": "{ASCM_HOST}addredec" },
            { "target": "adddoc", "desc": "Add Document", "source-uri": "{ASCM_HOST}adddoc" },
            { "target": "documents", "desc": "Documents", "source-uri": "{ASCM_HOST}documents" },
            { "target": "attachredecdoc", "desc": "Attach Redec Document", "source-uri": "{ASCM_HOST}attachredecdoc" },
            { "target": "job", "desc": "Job Trigger", "source-uri": "{JOB_HOST}job" },
            { "target": "task", "desc": "Task Trigger", "source-uri": "{JOB_HOST}task" }
        ]
    },
    {
        "Scenario": "Tasks",
        "Routes": [
            { "target": "instructions", "desc": "Instructions", "source-uri": "{DYNAMIC_HOST}instructions" }
        ]
    },
    {
        "Scenario": "Prompt",
        "Routes": [
      		 { "target": "sel", "desc": "Stable Export Limit", "source-uri": "{DYNAMIC_HOST}sel" },
            { "target": "mel", "desc": "Maximum Export Limit", "source-uri": "{DYNAMIC_HOST}mel" },
            { "target": "mnzt", "desc": "MNZT", "source-uri": "{DYNAMIC_HOST}mnzt" },
            { "target": "mzt", "desc": "MZT", "source-uri": "{DYNAMIC_HOST}mzt" }       
        ]
    },
    {
        "Scenario": "Daily",
        "Routes": [
    	     { "target": "sel", "desc": "Stable Export Limit", "source-uri": "{MARKET_HOST}sel" },
            { "target": "mel", "desc": "Maximum Export Limit", "source-uri": "{MARKET_HOST}mel" },
            { "target": "mnzt", "desc": "MNZT", "source-uri": "{MARKET_HOST}mnzt" },
            { "target": "mzt", "desc": "MZT", "source-uri": "{MARKET_HOST}mzt" }
         ]
    }
   ]

```


