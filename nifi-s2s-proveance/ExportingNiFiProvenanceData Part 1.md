## Extracting NiFi Provenance Data using SiteToSiteProvenanceReportingTask Part 1##

In this tutorial, we will learn how configure NiFi to send provenance data to a second NiFi instance:

1. Downloading NiFi
2. Configure Site-To-Site
3. Setting up the first Flow
4. Setting up the Provenance Reporting Instance Flow
5. Adding Site To Site Provenance Reporting
6. Starting the flow
7. Inspecting the Provenance data
8. Next Steps

**NOTE**: In this tutorial we are going to be taking the following shortcuts, in the spirit of understanding the concepts. Specifically we are going to:
  * Run two NiFi instances on the same host.
  * Not configure security in either NiFi instance or use a secure transport between NiFi hosts.

These are not the best practices that would be recommended in a production environment.

## References ##
For a primer on NiFi please refer to the [NiFi Getting Started Guide][1].

[1]: https://nifi.apache.org/docs/nifi-docs/html/getting-started.html


## Downloading and Configuring NiFi
1. Downlaod the latest version of NiFi from  [here](https://nifi.apache.org/download.html)
2. Extract the contents of the download, we will refer to this instance as the "first NiFi instance"
3. Copy the exatracted contents to a new directory, we will refer to this as the "Provenance Reporting Instance"


## Configuring the Provenance Reporting Instance NiFi
Before starting up the this NiFi instane we need to enable Site-to-Site communication so that it can receive the provenance data and also change the listening port for NiFi so ti does not conflict with the first instance. To do that do the following:

* Open <$PROVENANCE_REPORTING_INSTANCE_NIFI_INSTALL_DIR>/conf/nifi.properties in your favorite editor
* Change:

			nifi.remote.input.host=
			nifi.remote.input.socket.port=
			nifi.remote.input.secure=true
      nifi.web.http.port=8080
	To


			nifi.remote.input.host=localhost
			nifi.remote.input.socket.port=10000
			nifi.remote.input.secure=false
      nifi.web.http.port=8088

## Starting up the both instances of NiFi
We now have the two NiFi instances ready, to start them do the following:
1. Navigate to the directory for the first NiFi instance and start it according to your operting system
2. Navigate to the directory for the Provenance Reporting Instance and start it according to your operting system

## Setting up the first Flow for NiFi
Now that we have two NiFi instances up and running the next thing to do is to create our data flow. To keep things simple we are going to use one of the sample NiFi Dataflow Templates. In particular we are going to use the DateConversion flow which can be downloaded from [here](https://cwiki.apache.org/confluence/download/attachments/57904847/DateConversion.xml?version=2&modificationDate=1462288576000&api=v2)

After downloading this template, import it, and then create an instance of it. For instructions on how to import a template please see the [Template section of the NiFi user guide](http://127.0.0.1:8080/nifi-docs/html/user-guide.html#templates).  After creating an instance of the template your NiFi canvas should then look similar to this:

![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/date-conversion-flow.png>)
Figure 1. DateConversion Flow

I have modified the layout on my canvas so it easily fits on the screen.

## Setting up the Provenance Reporting Instance Flow
1.	Open a browser and go to the "Provenance Reporting Instance" instance: http://127.0.0.1:8088/nifi
2.  Create an input port called "Prov Data"
3.  Create a LogAttribute Processor
4.  Connect the input port to the LogAttribute Processor
5. Start the ProvData input port

Your flow should look similar to the following:


![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/provenance_flow.png>)
	Figure 2. Provenance Reporting Instance Flow

## Adding Site To Site Provenance Reporting
We are now ready to add the provenance reporting task the NiFi flow. To do this do the following:

1. Go to the "hamburger menu" in the top right of the UI and chose "Controller Settings"

    ![Display Name](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/controller_settings.png>)

2. Go the "Reporting Tasks" tab and click the ![plus](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/plus_button.png>) icon
3. Chose the SiteToSiteProvenanceReportingTask ![plus](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/s2s_task_add.png>)
4. Click on the pencil icon and edit the SiteToSiteProvenanceReportingTask properties so it looks like this: ![s2s properties](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/s2s_task_edit_properties.png>)

    **NOTE** I set the batch size to 1, this is for demo purposes only. In a production environment you would want to adjust this or leave it as the default 1000.
5. Adjust the settings for the SiteToSiteProvenanceReportingTask so that the run schedule is 5 seconds and not the default 5 minutes. ![s2s_settings](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/s2s_task_edit_run.png>)

    **NOTE** Again this is for demo purposes only. In a production environment you may want to leave this as the default or adjust it accordingly.

## Starting the flow
We are now all ready to start the DateConversion flow we created before. Go ahead and just click on the start button on the operate palette.

![operate_palette](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/operate_palette.png>)


## Inspecting the Provenance data
To inspect the provenance data, go the Provenance Reporting Instance instance (http://127.0.0.1:8088/nifi). With the LogAttribute processor stopped, you should see the flow files build up in the queue between the input port and the LogAttribute processor.

![operate](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/nifi-s2s-proveance/flow_queue.png>)

To view the provenenace data do the following:
1. Right click on the queue and chose "List queue"
2. Pick one of the flow files in the queue
3. Chose "View" to see the content, an example of a formatted provenance event looks like this:

```
[{
	"eventId": "07b4693a-20b1-4a4d-9dc3-37d4c8f93e59",
	"eventOrdinal": 0,
	"eventType": "CREATE",
	"timestampMillis": 1482171900667,
	"timestamp": "2016-12-19T18:25:00.667Z",
	"durationMillis": -1,
	"lineageStart": 1482171900657,
	"componentId": "3fde726d-5cc1-4bb6-9e06-35218a9c58a8",
	"componentType": "GenerateFlowFile",
	"componentName": "GenerateFlowFile",
	"entityId": "47160cde-d484-4292-be3d-476cd4fff1cb",
	"entityType": "org.apache.nifi.flowfile.FlowFile",
	"entitySize": 1024,
	"updatedAttributes": {
		"path": "./",
		"uuid": "47160cde-d484-4292-be3d-476cd4fff1cb",
		"filename": "19180888360764"
	},
	"previousAttributes": {},
	"actorHostname": "hw13095.attlocal.net",
	"contentURI": "http://hw13095.attlocal.net:8080/nifi-api/provenance-events/0/content/output",
	"previousContentURI": "http://hw13095.attlocal.net:8080/nifi-api/provenance-events/0/content/input",
	"parentIds": [],
	"childIds": [],
	"platform": "nifi",
	"application": "NiFi Flow"
}]
```
## Next Steps
Now that you have data flowing to your Provenenace Reporting NiFi instance, you can take that JSON data and send it to any number of destinations to do further analysis on it.
