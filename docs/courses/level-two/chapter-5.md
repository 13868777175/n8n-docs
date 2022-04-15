# Automating a business workflow

Remember [our friend Nathan](/courses/level-one/chapter-3)?

**Nathan 🙋:** Hello, it's me again. My manager was so impressed with my first workflow automation solution, that she entrusted me with more responsibility.<br/>
**You 👩‍🔧:** More work and responsibility. Congratulations, I guess. What do you need to do now?<br/>
**Nathan 🙋:** I got access to all sales data and am responsible for creating two reports, as PDF and spreadsheet files. They are based on data from different sources and come in different formats.<br/>
**You 👩‍🔧:** Sounds like a lot of manual work–but the kind that can be automated. So let's do it!


## Designing the workflow

Now that we have an idea of what Nathan wants to automate, let’s list the steps he needs to take to achieve this:

1. Get and combine data from all necessary sources.
2. Format the dates.
3. Write binary data (spreadsheet and PDF).
4. Send notifications via email and Discord.

n8n provides core nodes for all these steps. This use case is somewhat complex and it will be made up of three separate workflows:

1. A workflow that merges the company data with external information.
2. A workflow that calculates and generates the reports.
3. An error workflow that monitors the second (main) workflow.

Next, you will build these three workflows with step-by-step instructions.

## Building the workflow

### Workflow 1 – Fill in missing data

The company's customer data is stored in Airtable. It contains information about the customers' *ID*, *country*, *email*, and *join date*, but lacks data about their respective *region* and *subregion*. You need to fill in these last two fields in order to create the reports for regional sales.

To accomplish this task, you first need to make a copy of this [customers table](https://airtable.com/shrZBXHXWvQ57LHuX) in your Airtable account.

<iframe class="airtable-embed" src="https://airtable.com/embed/shrNX9tjPkVLABbNz?backgroundColor=orange&viewControls=on" frameborder="0" onmousewheel="" width="100%" height="533" style="background: transparent; border: 1px solid #ccc;"></iframe>

Next, you have to build a small workflow that merges data from Airtable and a REST API.

1. Use the [*Airtable node*](/integrations/nodes/n8n-nodes-base.airtable/) to list the data in the Airtable table named `customers`.
2. use the [*HTTP Request node*](/integrations/core-nodes/n8n-nodes-base.httpRequest/) to get data from the REST Countries API: `https://restcountries.com/v3.1/all`. This will return data about world countries. Note that the incoming data needs to be split into items.
3. Use the [*Merge node*](/integrations/core-nodes/n8n-nodes-base.merge/) to merge data from Airtable and the Countries API by country name (the common key), represented as `customerCountry` in Airtable and `name.common` in the Countries API, respectively.
4. Use the *Airtable node* to update the fields *region* and *subregion* in Airtable with the data from the Countries API.

The workflow should look like this:

<figure><img src="/_images/courses/level-two/chapter-five/workflow1.png" alt="" style="width:100%"><figcaption align = "center"><i>Workflow 1 for merging data from Airtable and the Countries API</i></figcaption></figure>


!!! question "Quiz questions"

    * How many items does the *HTTP Request node* return?
    * How many items does the *Merge node* return?
    * How many unique regions are assigned in the customers table?
    * What is the subregion assigned to the customerID 10?

### Workflow 2 – Aggregate data and create reports

**Part 1 - Getting data**

1. Use the *HTTP Request node* to get data from the API endpoint that stores company data. Configure the following node parameters:

      * Authentication: Header Auth
      * URL: https://internal.users.n8n.cloud/webhook/custom-erp
      * Options > Add Option > Split Into Items: toggle to true.
      * Headers > Add Header:
          * Name: unique_id
          * Value: your_unique_id

2. Use the *Airtable node* to list data from the `customers` table (where you updated the fields *region* and *subregion*).
3. Use the *Merge node* to merge data from the *Airtable* and *HTTP Request node*, based on the common key `customer ID`.

!!! question "Quiz questions"

    * How many items...

**Part 2 - Transforming data**

1. Use the *Items List node* to sort data by order price in descending order.
2. Use the *Split In Batches node* to split the data into batches of 2.
3. Use the *Set node* to set four values, referenced with expressions from the previous node: *customerEmail*, *customerLocation*, *customerSince*, and *orderPrice*.
4. Use the *Date & Time node* to change the date format to a different one.
5. Use the *Move Binary node* to transform the incoming data from JSON to binary format.

!!! question "Quiz questions"

    * How many items...

**Part 3 - Generating files**

9. Use the *Write Binary File node* to generate a PDF file.
10. Send the PDF file via email to an email address you have access to.
11. Use the *Discord node* to send a message in the n8n Discord channel `#course-level-two` with the Text: "I sent the PDF via email. My ID:" followed by your ID.


12. Use the *Spreadsheet File node* to create a spreadsheet with the file name set as the expression: `{{$runIndex > 0 ? 'file_low_orders':'file_high_orders'}}`.
13. Use the *Google Drive node* to upload the spreadsheet to Google Drive.
14. Use the *Discord node* to send a message in the n8n Discord channel `#course-level-two` with the Text: <br/>
    "I uploaded the spreadsheet with name `{file name}` and kind `{file kind}`. My ID:" followed by your ID. <br/>
	 Note that you need to replace the text in curly brackets `{}` with expressions that take the respective information from the *Google Drive node*.

<figure><img src="/_images/courses/level-two/chapter-five/workflow2.png" alt="" style="width:100%"><figcaption align = "center"><i>Workflow 2 for aggregating data and generating files</i></figcaption></figure>


### Workflow 3 – Create an error workflow

To accomplish this task, you have to create an *Error Workflow* that monitors the main workflow.

1. Add an *Error Trigger node* (and execute it as a test).
2. To the *Error Trigger node*, connect a *Discord node* and configure the fields:<br/>

	* Webhook URL: The URL that you received in the email from n8n when you signed up for this course.
	* Text: The workflow `{workflow name}` failed, with the error message: `{execution error message}`. Last node executed: `{name of the last executed node}`. Check this workflow execution here: `{execution URL}`.

		Note that you need to replace the text in curly brackets `{}` with expressions that take the respective information from the *Error Trigger node*.<br/>

3. Execute the *Discord node*.
4. Set the newly created workflow as *Error Workflow* for the main workflow.

The workflow should look like this:

<figure><img src="/_images/courses/level-two/chapter-five/workflow3.png" alt="" style="width:100%"><figcaption align = "center"><i>Workflow 3 for monitoring workflow errors</i></figcaption></figure>
