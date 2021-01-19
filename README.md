[![Build Status](https://dev.azure.com/sminozhenko/OneMoreActionQueueFramework/_apis/build/status/sminozhenko.OneMoreActionQueueFramework?branchName=main)](https://dev.azure.com/sminozhenko/OneMoreActionQueueFramework/_build/latest?definitionId=1&branchName=main)

<br />
<p align="center">
  <h3 align="center">OneMore Action Queue framework</h3>

  <p align="center">
    OneMore Action Queue framework for MS Dynamics 365 for Finance and Supply Chain Management (ex. Microsoft Dynamics 365 for Finance and Operations, Dynamics AX 7)
    <br />
    <a href="https://github.com/sminozhenko/OneMoreActionQueueFramework"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/sminozhenko/OneMoreActionQueueFramework/issues">Report Bug</a>
    ·
    <a href="https://github.com/sminozhenko/OneMoreActionQueueFramework/issues">Request Feature</a>
  </p>
</p>


<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installationwithsourcecodes">Installation with source codes</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#demo">Demo</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]][repo-url]

Microsoft Dynamics 365 for Finance and Supply Chain Management is a popular ERP-system. Its out-of-the-box functionality covers a wide range of different business processes. Also, the system provides a set of built-in tools for the developers to create new features and make integrations with 3rd party solutions.

The purpose of the framework is to extend the current set of built-in tools inside the system which can be useful for both developers and consultants during the solution implementation and support. Very often there is a requirement to automate one or more actions in the system after the previous process step is completed. For example, when the general ledger journal is created via the integration or manually created and approved by the user, it needs to be posted automatically.

Usually, in these situations, the project team needs to make customization which requires additional costs and continuous testing during the version updates. Another option is to use external tools like Power Platform or Azure solutions which increases the complexity of the solution architecture. OneMore Action Queue framework is a tool written in x++ that allows to configure and automate the processes with less or no coding. This is a clear advantage for projects with limited budgets and tight timetables.

The framework has the following capabilities:
* **Queues**: collect actions in one place (queue) and process them in a unified and controllable way
* **Chain processing**: automate the process steps inside the system by converting alerts and business events to actions
* **Generic REST endpoint for action**: trigger actions from outside the system via a REST endpoint

The framework supports several types of action processing:
1. **Manually**: action processing is started manually by the users
2. **Batch**: actions are processed in a scheduled batch job
3. **Syncronious**: actions are processed in real-time when they are created (only for the actions created via REST endpoint)
4. **Reliable asynchronous**: actions are processed asynchronously via a batch job
5. **Process automation**: action processing is started by the background process within the process automation framework

The framework has the following built-in features:
1. **Menu item handler**: actions can be processed with the existing operations (limited to RunBase and SysOperation based operation with a query as parameters)
2. **Custom action handler**: actions can be processed with the custom logic created by the developer
3. **Error handling**: review the details about action queue processing results
4. **Retries**: set up a number of retries and retry intervals for the failed actions
5. **JSON-to-JSON transformations**: transform original JSON message to action queue JSON message form with [JUST.net][just-net]

Commonly used resources that I find helpful are listed in the acknowledgments.


<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

OneMore Action Queue framework utilizes the standard process automation framework. The minimum required version of the system is **10.0.14**.

### Installation with source code on development VM

1. Open Command line tool as administrator on the development VM and switch to the `Service volume` disk (usually it's disk `K:\`)
2. Clone the repo
   ```sh
   git clone https://github.com/sminozhenko/OneMoreActionQueueFramework.git
   ```
3. Create a directory junction at `K:\AosService\PackagesLocalDirectory` that pointed to `K:\OneMoreActionQueueFramework\OneMoreActionQueueFramework` folder
   ```sh
   mklink /J K:\AosService\PackagesLocalDirectory\OneMoreActionQueueFramework K:\OneMoreActionQueueFramework\OneMoreActionQueueFramework
   ```
4. Open Visual studio and open VS solution `OneMoreActionQueueFramework.sln` located at `K:\OneMoreActionQueueFramework\OneMoreActionQueueFramework_Project`
5. Build the solution
6. Run DB synchronization


<!-- USAGE EXAMPLES -->
## Usage

### Use case: Post a general journal once it's approved in a workflow

Prerequisites: approval workflow for general journals set up and active and alert as business event is created when field Workflow approval status is changed to Approved

 https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/business-events/alerts-business-events

#### Implement journal posting via business events and standard multi-posting class

1. Create new action queue type record:
  a. Go to **System administration > OneMore action queue > Setup > Action queue type**
  b. Create a new record and set values in fields **Action type Id** and **Description**
  c. In the **Type** field, select an option **Menu item**
  d. In the **Processing type** field, select an option **Reliable asynchronous**
  e. Specify retry parameters in **Retry parameters** field group (optionally)
  f. Select the system administrator user that will be used to run posting in **Run as** field, otherwise posting will be executed under the user who created the record in the action queue
  g. Select **Batch group id** to restrict posting to certain batch group (optionally)
  h. Save the record

2. Set **Transformation JSON** to convert business event JSON message to action queue JSON message.
    ```
    {
    "DataAreaId": "#valueof($.DataAreaId)", "Parameters":[{"ParameterName": "JournalNum", "ParameterValue": "#valueof($.KeyValue1)"}]
    }
    ```

    More information about JSON transformation rules can be found in [JUST.net][just-net] documentation.

3. Set menu item parameters for action type:
	a. Press the button **Set menu item parameters**
	b. In **Set menu item parameters** dialog select **Post journal** action and press **OK** button:
![Set menu item parameters dialog Screen Shot][set-menu-item-parameters-post-journal-screenshot]
	c. In the next dialog mark **Late selection** checkbox and press **Select** button:
![Set menu item parameters dialog2 Screen Shot][set-menu-item-parameters-post-journal2-screenshot]
	d. In **Select** dialog add advanced range filter by **Journal batch number**:
    ```
    (OneMoreActionQueueParameterValue(JournalNum))
    ```

    ![Set menu item parameters dialog3 Screen Shot][set-menu-item-parameters-post-journal3-screenshot]

    **OneMoreActionQueueParameterValue** is special query range function which allows to read parameter value from JSON message in run-time.
	f. Finalize set up and close all dialogs by pressing **OK** button.

4. Activate created action type by enabling **Enabled** checkbox:
![Action type post journal Screen Shot][action-type-post-journal-screenshot]

5. Activate **BusinessEventsAlertEvent** for OneMore action queue endpoint:
	a. Go to **System administration > Business events > Business events catalog**
	b. Create new business event endpoint:
		* Select **OneMore action queue** as endpoint type and press **Next**.
		* On the next dialog provide endpoint name and select **Action type id** created in previous step:
      ![Business events endpoint post journal Screen Shot][business-events-endpoint-post-journal-screenshot]

    c. Activate **BusinessEventsAlertEvent** business event for newly created business event endpoint

6. Create and test an example:
	a. Create and submit a general journal for approval, approve it, and wait until the business event is created.
	b. Check that the business event is converted to action queue message and action is processed:
		* Go to **System administration > OneMore action queue > Setup > Action queue type**
		* Press **Action queue** button
		* In the **Action queue** form you can see all actions created for specific action type with the status, original JSON message, transformed JSON message and processing notes:
![Action queue Screen Shot][action-queue-screenshot]
#### Implement journal posting via the business events and custom action handler

When there is no standard operation that can be used with **Menu item** action type or the standard behavior should be changed, **Custom action** handler can be created.

1. Create new contract class which will represent JSON message for custom action. The new class should be inherited from `OneMoreActionQueueContract`:
  ```
  /// <summary>
  /// OneMore action queue ledger journal post contract
  /// </summary>
  [DataContract]
  class OneMoreActionQueueDemoLedgerJournalTablePostContract extends OneMoreActionQueueContract
  {
      private LedgerJournalId journalId;

      [DataMember("JournalId")]
      public LedgerJournalId parmJournalId(LedgerJournalId _journalId = journalId)
      {
          journalId = _journalId;
          return journalId;
      }

  }
  ```
2. Create new processor class for custom action to implement the general journal posting logic. Class should be inherited from class `OneMoreActionQueueProcessor` and decorated with attribute `OneMoreActionQueueProcessorAttribute` with contract class as parameter:
  ```
  /// <summary>
  /// OneMore action queue ledger journal post processor
  /// </summary>
  [OneMoreActionQueueProcessorAttribute(classStr(OneMoreActionQueueDemoLedgerJournalTablePostContract))]
  class OneMoreActionQueueDemoLedgerJournalTablePostProcessor extends OneMoreActionQueueProcessor
  {
      public void runActionInternal()
      {
          OneMoreActionQueueDemoLedgerJournalTablePostContract ledgerJournalTablePostContract = contract as OneMoreActionQueueDemoLedgerJournalTablePostContract;

          LedgerJournalTable ledgerJournalTable = LedgerJournalTable::find(ledgerJournalTablePostContract.parmJournalId());

          ttsbegin;

          LedgerJournalCheckPost ledgerJournalCheckPost = LedgerJournalCheckPost::newLedgerJournalTable(ledgerJournalTable, NoYes::Yes);
          ledgerJournalCheckPost.runOperation();

          ttscommit;
      }

  }
  ```

3. Build the solution.
4. Create a new action queue type record for the custom action:
	a. Go to **System administration > OneMore action queue > Setup > Action queue type**
	b. Create new record and set values in fields **Action type Id** and **Description**
	c. In the **Type** field, select an option **Custom action**
	d. Select processor class created in previous steps in the opened dialog:
![Action type post journal custom Screen Shot][action-type-post-journal-custom-screenshot]
	e. In the **Processing type** field, select an option **Reliable asynchronous**
	f. Specify retry parameters in **Retry parameters** field group (optionally)
	g. Select the system administrator user that will be used to run posting in **Run as** field, otherwise posting will be executed under the user who created the record in the action queue
	h. Select **Batch group id** to restrict posting to certain batch group (optionally)
	i. Save the record
5. Set **Transformation JSON** to convert business event JSON message to action queue JSON message:
    ```
    {
		"DataAreaId": "#valueof($.DataAreaId)", "JournalId" : "#valueof($.KeyValue1)"
	}
    ```
6. Activate created action type by enabling **Enabled** checkbox.
7. Activate **BusinessEventsAlertEvent** for OneMore action queue endpoint:
	a. Go to **System administration > Business events > Business events catalog**
	b. Create new business event endpoint:
		* Select **OneMore action queue** as endpoint type and press **Next**.
		* On the next dialog provide endpoint name and select **Action type id** created in previous step.

    c. Activate **BusinessEventsAlertEvent** business event for newly created business event endpoint

8. Create and test an example as described for **Menu item** type:
![Action queue custom Screen Shot][action-queue-custom-screenshot]
### Use case: Post a general journal from the external system

For this example you need to create a similar action type record as created in [Implement posting via business events and standard multi-posting class](#implement-posting-via-business-events-and-standard-multi-posting-class) case, but you don't need to set JSON transformation and set up business events.

The framework has a generic REST API endpoint to create the actions in the system.

Postman tool will be used to send action to D365FO and simulate the external system. More information about how to set up Postman with D365FO can be found in the documentation:
https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/data-entities/third-party-service-test

To push action to D365FO, we need to send POST request to the address:
`https://host_url/api/services/OneMoreActionQueueServiceGroup/OneMoreActionQueueService/RunActionWithActionId`

with body:

```
{
	"_actionId": "{{$guid}}",
	"_typeId": "PostGeneralLedgerJournalPostman",
	"_actionMessage": "{\"DataAreaId\":\"usmf\",\"Parameters\":[{\"ParameterName\":\"JournalNum\",\"ParameterValue\":\"00636\"}]}"
}
```

_actionId - GUID value for new action
_typeId - Identifier for action queue type record
_actionMessage - actual JSON message. Note, that double-quotes should be escaped.

OData response contains 2 fields:

  * ResponseType: indicates status of action processing (OK, Warning, Error)
  * Notes: processing notes.

Example of the OK response:

```
{
    "ResponseType": "OK",
    "Notes": ""
}
```

Example of error response:

```
{
    "ResponseType": "Error",
    "Notes": "Action '{023AA40E-B714-42A0-9D8F-BB8DFC123632}'. Post journals. Journal 00637 cannot be posted because the journal has been submitted for workflow approval and it is not approved.
    Action '{023AA40E-B714-42A0-9D8F-BB8DFC123632}'. Post journals. Validation errors occurred with journal batch number 00637, see the message center for more details.
    Action '{023AA40E-B714-42A0-9D8F-BB8DFC123632}'. Post journals. Journal 00636 has already been posted"
}
```

REST API endpoint is the only approach that supports **Synchronous** processing type. Note, that **Synchronous** processing type can cause delays in processing when mass amount of actions should be executed.

_For more examples, please refer to the [Documentation][repo-url]_

## Demo

The repository contains the demo model with classes described in usage section.

Run the next steps to install the model (framework model should be installed already):

1. Open Command line tool as administrator on the development VM and switch to the directory where repository was cloned (assume it's `K:\OneMoreActionQueueFramework`).
2. Create a directory junction at `K:\AosService\PackagesLocalDirectory` that pointed to `K:\OneMoreActionQueueFramework\OneMoreActionQueueFrameworkDemo` folder
   ```sh
   mklink /J K:\AosService\PackagesLocalDirectory\OneMoreActionQueueFrameworkDemo K:\OneMoreActionQueueFramework\OneMoreActionQueueFrameworkDemo
   ```
3. Open Visual studio and open VS solution `OneMoreActionQueueFrameworkDemo.sln` located at `K:\OneMoreActionQueueFramework\OneMoreActionQueueFrameworkDemo_Project`
4. Build the solution
5. Run DB synchronization

<!-- ROADMAP -->
## Roadmap

See the [open issues][issues-url] for a list of proposed features (and known issues).

<!-- CONTRIBUTING -->
## Contributing

Contributions are what makes the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request



<!-- LICENSE -->
## License

Distributed under the MIT License. See [`LICENSE`][license-url] for more information.



<!-- CONTACT -->
## Contact

Sergei Minozhenko - sminozhenko@gmail.com
[![LinkedIn][linkedin-shield]][linkedin-url]


Project Link: [https://github.com/sminozhenko/OneMoreActionQueueFramework][repo-url]



<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
* [JUST.net][just-net]


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[repo-url]: https://github.com/sminozhenko/OneMoreActionQueueFramework.git
[issues-url]: https://github.com/sminozhenko/OneMoreActionQueueFramework/issues
[license-url]: https://github.com/sminozhenko/OneMoreActionQueueFramework/blob/main/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/sergei-minozhenko-09185870/
[just-net]: https://github.com/WorkMaze/JUST.net
[product-screenshot]: Images/screenshot.png
[set-menu-item-parameters-post-journal-screenshot]: Images/set_menu_item_parameters_post_journal.png
[set-menu-item-parameters-post-journal2-screenshot]: Images/set_menu_item_parameters_post_journal2.png
[set-menu-item-parameters-post-journal3-screenshot]: Images/set_menu_item_parameters_post_journal3.png
[action-type-post-journal-screenshot]: Images/action_type_post_journal.png
[business-events-endpoint-post-journal-screenshot]: Images/business_events_endpoint_post_journal.png
[action-queue-screenshot]: Images/action_queue.png
[action-type-post-journal-custom-screenshot]: Images/action_type_post_journal_custom.png
[action-queue-custom-screenshot]: Images/action_queue_custom.png