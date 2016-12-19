## Extracting NiFi Provenance Data using SiteToSiteProvenanceReportingTask Part 1##

In this tutorial, we will learn how configure NiFi to send provenance data to a second NiFi instance:

1. Downloading NiFi
2. Configure Site-To-Site
3. Setting up the first Flow
4. Setting up the Provenance Reporting Instance Flow
5. Enjoying the data flow!

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
3.

## Configuring the Provenance Reporting Instance NiFi
Before starting up the this NiFi instane we need to enable Site-to-Site communication so that it can receive the provenance data and also change the listening port for NiFi so ti does not conflict with the first instance. To do that do the following:

* Open <$NIFI_INSTALL_DIR>/conf/nifi.properties in your favorite editor
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

![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/NiFi_Clean.png>)
Figure 2. DateConversion Flow

I have modified the layout on my canvas so it easily fits on the screen.
1.	Open a browser and go to: http://\<nifi-host>:\<port>/nifi on my machine that url looks is http://127.0.0.1:8080/nifi and going to it in the browser looks like this:

	![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/NiFi_Clean.png>)
	Figure 2. Empty NiFi Canvas

2.	.

	![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/InputPort.png>)
	Figure 3. Generate Flow File

3. Attriubutes to JSON.

	![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/LogAttribute.png>)
	Figure 4. Attributes to JSON processor

4.	Put File.

  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/nifi-flow.png>)
  Figure 5. NiFi Flow

5.  We are now ready to build the MiNiFi side of the flow. To do this do the following:
	* Start the From MiNiFi Input Port
	* Add a GenerateFlowFile processor to the canvas (don't forget to configure the properties on it)
	* Add a Remote Processor Group to the canvas as shown below in Figure 6

  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/AddingRPG.png>)

  Figure 6. Adding the Remote Processor Group

   * For the URL copy and paste the URL for the NiFi UI from your browser
   * Connect the GenerateFlowFile to the Remote Process Group as shown below in figure 7. (You may have to refresh the Remote Processor Group, before the input port will be available)

  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/AddingGFFToRPGConnection.png>)

  Figure 7. Adding GenerateFlowFile Connection to Remote Processor Group

6.  Your canvas should now look similar to what is shown below in figure 8.

  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/WholeFlow.png>)

  Figure 8. Adding GenerateFlowFile Connection to Remote Processor Group

7. The next step is to generate the flow we need for MiNiFi. To do this do the following steps:
  * Create a template for MiNiFi illustrated below in figure 9.
  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/CreatingTemplate.png>)
  Figure 9. Creating a template

  *   Select the GenerateFlowFile and the NiFi Flow Remote Processor Group (these are the only things needed for MiMiFi)
  *   Select the "Create Template" button from the toolbar
  *   Choose a name for your template

8. We now need to save our template, as illustrated below in figure 10.
  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/TemplateButton.png>)

  Figure 10. Template button

9. Now we need to download the template as shwon below in figure 11
  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/DownloadTemplate.png>)

  Figure 11. Saving a template

10.  We are now ready to setup MiNiFi. However before doing that we need to convert the template to YAML format which MiNiFi uses. To do this we need to do the following:
  * Navigate to the minifi-toolkit directory (minifi-toolkit-0.0.1)
  * Transform the template that we downloaded using the following command:

      ````bin/config.sh transform <INPUT_TEMPLATE> <OUTPUT_FILE>````

    For example:

      ````bin/config.sh transform MiNiFi_Flow.xml config.yml````



You should be able to now go to your NiFi flow and see data coming in from MiNiFi.
