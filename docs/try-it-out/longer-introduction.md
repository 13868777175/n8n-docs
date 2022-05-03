# A slightly longer introduction

This guide shows you how to automate a task using a workflow in n8n, explaining key concepts along the way. You will:

* Install n8n's desktop app.
* Create a workflow from scratch. You'll build a workflow that runs every week to get data from NASA, filters the data, and generates two reports.
* Understand key concepts and skills, including:
    * Starting workflows with trigger nodes
    * Configuring credentials
    * Manipulating data
    * Representing logic in an n8n workflow
    * Using expressions

<n8n-demo workflow='{
  "name": "Quickstart test",
  "nodes": [
    {
      "parameters": {},
      "name": "Start",
      "type": "n8n-nodes-base.start",
      "typeVersion": 1,
      "position": [
        240,
        300
      ]
    },
    {
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "mode": "custom",
              "cronExpression": "0 9 * * 1"
            }
          ]
        }
      },
      "name": "Cron",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [
        420,
        620
      ]
    },
    {
      "parameters": {
        "resource": "donkiSolarFlare",
        "additionalFields": {
          "startDate": "={{$today.plus({days: -7}).toFormat(\"yyyy-MM-dd\")}}"
        }
      },
      "name": "NASA",
      "type": "n8n-nodes-base.nasa",
      "typeVersion": 1,
      "position": [
        640,
        620
      ],
      "credentials": {
        "nasaApi": {
          "id": "5",
          "name": "NASA account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json[\"classType\"]}}",
              "operation": "regex",
              "value2": "[ABC]"
            }
          ]
        }
      },
      "name": "IF",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [
        860,
        620
      ]
    },
    {
      "parameters": {
        "requestMethod": "POST",
        "url": "https://www.toptal.com/developers/postbin/1651063625300-2016451240051",
        "responseFormat": "string",
        "options": {},
        "bodyParametersUi": {
          "parameter": [
            {
              "name": "flareSize",
              "value": "=There was a solar flare of class {{$json[\"classType\"]}}"
            }
          ]
        }
      },
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 1,
      "position": [
        1080,
        400
      ]
    },
    {
      "parameters": {
        "requestMethod": "POST",
        "url": "https://www.toptal.com/developers/postbin/1651063625300-2016451240051",
        "responseFormat": "string",
        "options": {},
        "bodyParametersUi": {
          "parameter": [
            {
              "name": "flareSize",
              "value": "=There was a solar flare of class {{$json[\"classType\"]}}"
            }
          ]
        }
      },
      "name": "HTTP Request1",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 1,
      "position": [
        1080,
        760
      ]
    },
    {
      "parameters": {
        "content": "# Test"
      },
      "name": "Note",
      "type": "n8n-nodes-base.stickyNote",
      "typeVersion": 1,
      "position": [
        440,
        440
      ]
    }
  ],
  "connections": {
    "Cron": {
      "main": [
        [
          {
            "node": "NASA",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "NASA": {
      "main": [
        [
          {
            "node": "IF",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "IF": {
      "main": [
        [
          {
            "node": "HTTP Request",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "HTTP Request1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {},
  "id": 16
}'></n8n-demo>


## Step one: Install and run n8n

--8<-- "_snippets/try-it-out/install-run-n8n.md"

## Step two: Create a new workflow

Create a blank workflow:

1. On the **Workflow templates** screen, select **New blank workflow**. n8n opens the new workflow.
2. Rename the workflow to something meaningful, such as **Quickstart**. Select the current workflow name, and replace it.

## Step three: Add a trigger node

n8n provides two ways to start a workflow:

* Manually, by selecting **Execute workflow**, or from the CLI if you installed n8n with npm or Docker.
* Automatically, using a trigger node as the first node. The trigger node runs the workflow in response to an external event, or based on your settings.

For this tutorial, use the [Cron trigger](/integrations/core-nodes/n8n-nodes-base.cron/). This allows you to run the workflow on a schedule:

1. Select **Add node** <span class="inline-image">![Add node icon](/_images/try-it-out/quickstart/add-node.png)</span>.
2. Search for **Cron**. n8n shows a list of nodes that match the search.
3. Select **Cron** to add the node to the canvas. n8n opens the node.
4. Select **Add Cron Time**.
5. For **Mode**, select **Custom**.
6. In **Cron Expression**, enter the following cron expression: `0 9 * * 1`. If you need to generate your own cron expressions, [crontab guru](https://crontab.guru/) can help. The expression in this example means the workflow runs at 09:00 every Monday.
7. Close the node view to return to the canvas.


## Step four: Add the NASA node and set up credentials

The [NASA node](/integrations/nodes/n8n-nodes-base.nasa/) allows you to interact with NASA's [public APIs](https://api.nasa.gov/). The API gives you data to work with in this tutorial.

1. Select the **Add node** connector <span class="inline-image">![Add node icon](/_images/try-it-out/quickstart/add-node-small.png)</span> on the Cron node.
2. Search for **NASA**. n8n shows a list of nodes that match the search.
3. Select **NASA** to add the node to the canvas. n8n opens the node.
4. To access the NASA APIs, you need to set up credentials:
    1. Select the  **Credential for NASA API** dropdown.
    2. Select **- Create New -**. n8n opens the credentials view.
    3. Go to [NASA APIs](https://api.nasa.gov/) and fill out the form in **Generate API Key**. NASA generates the key and displays it.
    4. Copy the key, and paste it into **API Key**.
    5. Select **Save**.
    6. Close the credentials screen. n8n returns to the node. The new credentials should be automatically selected in **Credential for NASA API**.
5. In **Resource**, select **DONKI Solar Flare**. This resource returns a report about recent solar flares.
6. By default, DONKI Solar Flare provides data for the past 30 days. To limit it to just the last week, use **Additional Fields**:
    1. Select **Add field**.
    2. Select **Start date**.
    3. To get a report starting from a week ago, you can use an expression: next to **Start date**, select **Parameter options** <span class="inline-image">![Parameter options icon](/_images/try-it-out/quickstart/parameter-options.png)</span> > **Add Expression**. n8n opens the **Edit Expression** modal.
    4. In the **Expression** field, enter the following expression:
    ```js
    {{$today.plus({days: -7}).toFormat("yyyy-MM-dd")}}
    ```
    This generates a date in the correct format, seven days before the current date.

    !!! note "Date and time in n8n"
        n8n uses Luxon to work with date and time, and also provides two variables for convenience: `$now` and `$today`. For more information, refer to [Expressions > Luxon](/code-examples/expressions/luxon/). 

7. You can now check that the node is working and returning the expected date: select **Execute node** to run the node manually. n8n calls the NASA API and displays details of solar flares in the past seven days in the **OUTPUT** section.

## Step five: Add logic with the If node

n8n supports complex logic in workflows. In this tutorial, use the [If node](/integrations/core-nodes/n8n-nodes-base.if) to create two branches that each generate a report from the NASA data. Solar flares have five possible classifications. You'll create logic that sends a report with the lower classifications to one output, and the higher classifications to another. 

Add the If node:

1. Select the **Add node** connector <span class="inline-image">![Add node icon](/_images/try-it-out/quickstart/add-node.png)</span> on the NASA node.
2. Search for **If**. n8n shows a list of nodes that match the search.
3. Select **If** to add the node to the canvas. n8n opens the node.
4. Select **Add condition** > **String**.
5. You need to check the value of the `classType` propperty in the NASA data. To do this:
    1. Next to **Value 1**, select **Parameter options** <span class="inline-image">![Parameter options icon](/_images/try-it-out/quickstart/parameter-options.png)</span> > **Add Expression**. n8n opens the expressions editor for this field.
    2. Select **Current Node** > **Input Data** > **JSON** > **classType**. n8n adds the expression to the **Expression** editor, and displays a sample output.
    3. Close the expressions editor to return to the node.
    4. In **Operation**, select **Regex Match**.
    5. In **Value 2**, enter **[ABC]**.

    !!! note "Regex"
        Regex is short for regular expression. A [regular expression](https://en.wikipedia.org/wiki/Regular_expression) is a sequence of characters that match a pattern. To create your own regex patterns, try [regex101](https://regex101.com/).

6. You can now check that the node is working and returning the expected date: select **Execute node** to run the node manually. n8n tests the data against the condition, and shows which results match true or false in the **OUTPUT** panel.

## Step six: Output data from your workflow [TODO: refactor this when we have the Postbin node]

The last step of the workflow is to send the two reports about solar flares. For this example, you'll send data to [Postbin](https://www.toptal.com/developers/postbin/). Postbin is a service that receives data and displays it on a temporary web page. 

1. On the If node, select the **Add node** connector <span class="inline-image">![Add node icon](/_images/try-it-out/quickstart/add-node.png)</span> labelled **true**.
2. Search for **HTTP Request**. n8n shows a list of nodes that match the search.
3. Select **HTTP Request** to add the node to the canvas.
4. Change **Request Method** to **Post**.
5. Go to [Postbin](https://www.toptal.com/developers/postbin/) and select **Create Bin**.
6. Copy the POST url from the curl example. It looks similar to `https://www.toptal.com/developers/postbin/1651063625300-2016451240051`.
7. In n8n, paste your Postbin URL into **URL**.
8. Change **Response Format** to **String**.
9. Now, configure the data to send to Postbin. In **Body Parameters**, select **Add Parameter**.
10. In **Name**, enter **flareSize**.
11. For the **Value**, use an expression to get the exact solar flare classification and write a message: 
    1. Select **Parameter options** <span class="inline-image">![Parameter options icon](/_images/try-it-out/quickstart/parameter-options.png)</span> > **Add Expression**. n8n opens the expressions editor.
    2. Select **Current Node** > **Input Data** > **JSON** > **classType**. n8n adds the expression to the **Expression** editor, and displays a sample output.
    3. The expression is: `{{$json["classType"]}}`. Add a message to it, so that the full expression is:
    ```js
    {{"There was a solar flare of class " + $json["classType"]}}
    ```
    4. Close the expressions editor to return to the node.
12. Close the HTTP Request node to return to the canvas.
13. Add another HTTP Request node, to handle the **false** output path from the If node:
    1. Hover over the HTTP Request node, then select **Copy** to duplicate the first HTTP Request node.
    2. Drag the **false** connector from the If node to the left side of the new HTTP Request node.
14. You can now test the entire workflow. Select **Execute Workflow**. n8n runs the workflow, showing each stage in progress.
15. Go back to your Postbin bin. Refresh the page to see the output.
16. If you want to use this workflow (in other words, if you want it to run once a week automatically), you need to activate it by selecting the **Active** toggle.

!!! note "Time limit"
    Postbin's bins exist for 30 minutes after creation. You may need to create a new bin and update the URL in the HTTP Request nodes, if you exceed this time limit.


## Next steps

* Take n8n's [courses](/courses/).
* Explore [hosting options](/hosting/options/).
* Check out example workflows: the [n8n blog](https://n8n.io/blog/tag/tutorial/) contains tutorials around popular examples, while the workflow templates gives you a community-sourced library of inspiration and starting points.