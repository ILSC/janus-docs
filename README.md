# Table of Contents
* [Introduction](#janus)
   * [Process Flow](#process-flow)
* [Setup](#setup)
   * [Installation](#installation)
   * [Configuration in P6](#configuration-in-p6)
   * [Configuration in Agile](#configuration-in-agile)
* [Startup and Shutdown](#startup-and-shutdown)
* [Usage](#usage)
   * [Linking Document to Deliverable](#linking-document-to-deliverable)
   * [Creating Deliverable in P6](#creating-deliverable-in-p6)
   * [Setting up Sync Schedule](#setting-up-sync-schedule)
   

# Janus
Janus provides integration between Primavera P6 and Agile PLM. Primavera is a Project management system and helps users to plan and monitor their projects. Users can setup a Project schedule capturing Activities, Resources, Deliverables and risks etc. related to a Project. Agile PLM on the other hand is Engineering centeric system providing users to templatize workflows and processes required to create and release engineering deliverables.

Janus provides a way for Project managers to track progress of engineering deliverables in context of the overall schedule of their project. For engineering teams it provides ability to priortize release of engineering drawings in line with the Project schedule setup by Project managers.

## Process Flow

The following figure depicts typical flow of information between Primavera and Agile enabled by Janus.

![Picture1.png](https://bitbucket.org/repo/Ag4kgL5/images/205842174-Picture1.png)

The deliverables in Project are linked to respective drawings or documents in Agile. In case the deliverables in P6 are linked top an activities in Project schedule the due date based on the activity dates for the deliverables is updated back in Agile.

As the drawing or document in Agile moves through the workflow, the status of the same is updated back on the respective deliverable in Project. Users can choose to setup the drawing or documents in Agile first and create the corresponding deliverable in P6 against the same using Janus or map the drawing or documents in agile to existing deliverables in P6 Project. 

# Setup

## Installation
Janus is delivered as a self contained executable jar or war file and can be run standalone or deployed on an application server like Tomcat or WebLogic. Oracle JRE 1.8.0_131+ must already be installed on the server where you want to setup Janus. Mongo DB must also be setup before you install Janus.


> Note: In case you are running containers in your organization, Janus is available as a Docker container as well. Contact Insight support for details on the same.

This manual lists down steps on how to set the same up as a standalone process. In case you need to deploy the same on Tomcat follow the usual steps you follow to deploy any other war file on Tomcat. For deployment on top of Weblogic you should contact Insight Support team to help you to setup the same.

To setup as a standalone process, create a directory you want to install the application to on your file system and copy the jar/war file to the same location along with the startup script relevant to your environment.

Edit the startup script (`startJanus.sh` or `startJanus.cmd`) and point the JAVA_HOME environment variable to the location of your JRE installation.

```
For windows: set JAVA_HOME=C:\Program File\Java\jre1.8.0_131
```
```
For Linux/Unix: export JAVA_HOME=/usr/java/jre1.8.0_131
```

You also need to update the configuration file `application.yml` to point to the Primavera P6, Agile PLM and Mongo DB environments you want to connect to.

To update P6 Environment information update the following section in the `application.yml` file.

```
p6:
  host: xxxxxxxxxxxxxx #Host running P6API application
  rmiPort: 9099 #RMI port setup in Primavera Configuration
  dbInstance: xxxxxx #Database instance for Primavera you want to connect to
  loginId: admin #User name for the account used to connect to P6 
  password: xxxxxxxxx # Password for the account used to login into P6
  filterby:
    projectcode:
      name: Active Projects # Project Code used to filer projects made available to users in Janus
      codeVal: Active # Project Code Value used to filer projects made available to users in Janus

```

To update Agile Environment information update the following section in the `application.yml` file.

```
agile:
  url: http://xxxxxxxx:xxxx/Agile # Agile application server url
  username: admin # User name for the account used to connect to Agile
  password: xxxxxxxx # Password for the account used to login into Agile
```

To update Mongo DB information update the following section in the `application.yml` file.

```
spring:
  data:
    mongodb:
      uri: mongodb://xxxxxxxxx:27017/janus  # Mongo database connection url
```
This completes the steps required to setup Janus. Additional configuration related to logging or embedded application server is optional. Description of these options is provided in additional configuration section.
## Configuration in P6
Janus requires a minimal configuratiun on P6 side. All you need to do is setup a `Project Code` that will be used to filter projects that should be available to users in PLM for mapping deliverables. This is the same `Project Code` that is setup in `application.yml` config file under `p6->filterby->projectcode` option as described in the previous section.

Rest of the configuration items like UDFs Document Status etc are created on the fly by Janus.

To create project code Go to Project Codes section under Enterprise Data in P6 and use the Add Code option. Refer Primavera Admin manual for more details related to the same.
![Picture2.png](https://bitbucket.org/repo/Ag4kgL5/images/1515524858-Picture2.png)

## Configuration in Agile
In Agile you need to setup certain attributes on **Page Two** of **Documents class**. These attributes are used by Janus to store mapping information related to the P6 Deliverable on Agile PLM document.

You need to use Agile Java Client to do the same. Refer Agile Admin manual for more details on setting up the same. You may also need to setup read and modify privileges related to these attributes. Ideally these should only be editable using the Agile login used in Janus.
![Picture3.png](https://bitbucket.org/repo/Ag4kgL5/images/4286225787-Picture3.png)

Following is the list of attributes that need to be setup in Agile.

Name | API Name| Data Type
---- | ------- | ---------
Primavera Project Id | PrimaveraProjectId | Text
WP Document Id | WPDocId | Text
Due Date | DueDate| Date

In addition to these you need to setup a URL Process Extension in Agile as well. This enables the user to initiate deliverable mapping from Agile drawing or document. This also need to be setup using Agile Java Client. The url in the url px should be pointing to `mapproject` endpoint of Janus application. The structure of the url will be as follows.
```
http://<Janus server>:<port>/mapproject
```

![Picture9.png](https://bitbucket.org/repo/Ag4kgL5/images/951691176-Picture9.png)

# Startup and Shutdown
To start Janus run the staruJanus.sh or startJanus.cmd script. Janus must only be started once the Primavera, Agile and Mongo DB we are using in Janus are already up and running.

A message similar to the one below on the terminal or prompt window will confirm startup of Application.
```
insight.plmconnect.Application   : Started Application in 14.487 seconds (JVM running for 15.31)
```

To shutdown Janus use the stopJanus.sh or stopJanus.cmd script.

# Usage
This section lists downs instructions on how to use Janus to sync up project deliverables.

## Linking Document to Deliverable
To link an Agile Document to an existing Primavera Project Deliverable, user can go to the Actions button on the Document in Agile and click on the `Link to Project Deliverable` option. This will open up a simple form in a pop up window and you can select the Project and an existing Work Product setup in the project and click on `Save Mapping` button.
![Picture4.png](https://bitbucket.org/repo/Ag4kgL5/images/2173466455-Picture4.png)

## Creating Deliverable in P6
To create a new deliverable in P6 based on a document in Agile, user needs to click on `Link to Project Deliverable` option under Action Button only but need to click on `New WP Document` button instead.

This will open up a `Create WP Document` form and user needs to select the project under which the document need to be created and click on `Create Document` Button.
![Picture5.png](https://bitbucket.org/repo/Ag4kgL5/images/2316530974-Picture5.png)

## Setting up Sync Schedule
The information related to deliverables mapped between the two systems in synced up at a frequency defined in the Sync Jobs setup by Janus admin.

Sync Jobs identify projects that need to be synchronised along with the schedule and frequency of the same. To setup sync jobs admin user needs to login into Janus and setup a Job.

Existing Jobs setup in Janus are visible to admin user at home page itself and admin can edit or remove these jobs or setup a new sync job.
![Picture6.png](https://bitbucket.org/repo/Ag4kgL5/images/3812540147-Picture6.png)   

You can setup single sync job for all projects or multiple jobs to each targeting a set of projects. A job definition includes Projects included in the Sync Job along with a Schedule for the same.

You can lookup Projects by Name or by filtering them using Project Code Assignments and then select the ones you need to include from the search results.
![Picture7.png](https://bitbucket.org/repo/Ag4kgL5/images/1943034790-Picture7.png)

To setup schedule for the Job, admin need to define the cron trigger under schedule tab and also give the Job a name and then click on `Schedule Job` button.
![Picture8.png](https://bitbucket.org/repo/Ag4kgL5/images/1834552688-Picture8.png)
