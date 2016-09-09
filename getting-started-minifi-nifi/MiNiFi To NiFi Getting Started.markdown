## Getting started with MiNiFi ##

In this tutorial, we will learn how configure MiNiFi to send data to NiFi:

1. Installing HDF
* Installing  MiNiFi
* Setting up the Flow for NiFi
* Setting up the Flow for MiNiFi
* Preparing the flow for MiNiFi
* Configuring and starting MiNiFi
* Enjoying the data flow!

## References ##
For a primer on HDF, you can refer to the tutorials here [Tutorials] [1] and [User Guides][2]

[1]: http://hortonworks.com/hadoop-tutorial/learning-ropes-apache-nifi/	"Tutortials"
[2]: http://hortonworks.com/products/data-center/hdf/ "User Guides"

## Installing HDF
1. If you do not have NiFi installed, please follow the instructions found [here](http://hortonworks.com/hadoop-tutorial/learning-ropes-apache-nifi/#section_3)

## Installing MiNiFi
Now that you have NiFi up and running it is time to download and install MiNiFi.

1. Open a browser. Download the MiNiFi Binaries from [Apache MiNiFi Downloads Page](http://nifi.apache.org/minifi/download.html). There are two options: tar.gz a format tailored to Linux and a zip file more compatible with Windows. If you are using a Mac either option is just fine.

 ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/Apache_NiFi_MiNiFi_Downloads.png>)

	Figure 1. MiNiFi download page

	For this tutorial I have downloaded the tar.gz on a Mac as shown above in

2. To install MiNiFi, extract the files from the compressed file to a location in which you want to run the application. I have chosen to install it to ```/Users/apsaltis/minifi-to-nifi```

	The image below show MiNiFi downloaded and installed in this directory:

 ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/MiNiFi_Install.png>)


## Setting up the Flow for NiFi
**NOTE:** Before starting NiFi we need to enable Site-to-Site communication. To do that do the following:

* Open <$NIFI_INSTALL_DIR>/conf/nifi.properties in your favorite editor
* Change:

			nifi.remote.input.socket.host=
			nifi.remote.input.socket.port=
			nifi.remote.input.secure=true

	To


			nifi.remote.input.socket.host=localhost
			nifi.remote.input.socket.port=10000
			nifi.remote.input.secure=false

* Restart NiFi if it was running

Now that we have NiFi up and running and MiNiFi installed and ready to go, the next thing to do is to create our data flow. To do that we are going to first start with creating the flow in NiFi. Remember if you do not have NiFi running execute the following command:

		<$NIFI_INSTALL_DIR>/bin/nifi.sh start


Now we should be ready to create our flow. To do this do the following:

1.	Open a browser and go to: http://\<nifi-host>:\<port>/nifi on my machine that url looks is http://127.0.0.1:8080/nifi and going to it in the browser looks like this:

	![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/NiFi_Clean.png>)
	Figure 2. Empty NiFi Canvas

2.	The first thing we are going to do is setup an Input Port. This is the port that MiNiFi will be sending data to. To do this drag the Input Port icon to the canvas and call it "From MiNiFi" as show below in figure 3.

	![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/InputPort.png>)
	Figure 3. Adding the Input Port

3. Now that the Input Port is configured we need to have somewhere for the data to go once we receive it. In this case we will keep it very simple and just log the attributes. To do this drag the Processor icon to the canvas and choose the LogAttribute processor as shown below in figure 4.

	![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/LogAttribute.png>)
	Figure 4. Adding the LogAttribute processor

4.	Now that we have the input port and the processor to handle our data, we need to connect them. After creating the connection your data flow should look like figure 5 below.

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

The next step is to generate the flow we need for MiNiFi. To do this do the following steps:
*   Create a template for MiNiFi illustrated below in figure 9.
  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/CreatingTemplate.png>)
  Figure 9. Creating a template

  *   Select the GenerateFlowFile and the NiFi Flow Remote Processor Group
  *   Select the "Create Template" button from the toolbar
  *   Choose a name for your template


*   We now need to save our template, as illustrated below in figure 10.
  ![<Display Name>](<https://raw.githubusercontent.com/apsaltis/hcc-assets/master/getting-started-minifi-nifi/TemplateButton.png>)
  Figure 10. Template button

*
