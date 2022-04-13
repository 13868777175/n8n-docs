# Understanding the data structure

In this chapter, you will learn about the data structure of n8n and how to use the [*Function node*](/integrations/nodes/n8n-nodes-base.function/) to transform data and simulate node outputs.


## Data structure of n8n

In a basic sense, n8n nodes function as an Extract, Transform, Load (ETL) tool. The nodes allow you to access (extract) data from multiple disparate sources, modify (transform) that data in a particular way, and pass (load) it along to where it needs to be.

The data that moves along from node to node in your workflow must be in a format (structure) that can be recognized and interpreted by each node. In n8n, this required structure is an **array of objects**.

!!! note "What is an array of objects?"

	An **array** is a list of values. The array can be empty or contain several elements. Each element is stored at a position (index) in the list, starting at 0, and can be referenced by the index number. For example, in the array `["Leonardo", "Michelangelo", "Donatello", "Raphael"];` the element `Donatello` is stored at index 2.

	An **object** stores key-value pairs, instead of values at numbered indexes as in arrays. The order of the pairs is not important, as the values can be accessed by referencing the key name. For example, the object below contains two properties (`name` and `color`):

	```
	{
		name: 'Michelangelo',
		color: 'blue',
	}
	```

	An **array of objects** is an array that contains one or more objects. For example, the array `turtles` below contains four objects:

	```
	var turtles = [
		{
			name: 'Michelangelo',
			color: 'orange',
		},
		{
			name: 'Donatello',
			color: 'purple',
		},
		{
			name: 'Raphael',
			color: 'red',
		},
		{
			name: 'Leonardo',
			color: 'blue',
		}
	];
	```

	You can access the properties of an object using **dot notation** with the syntax `object.property`. For example, `turtles[1].color` gets the color of the second turtle.

Data sent from one node to another is sent as an array of JSON objects. The elements in this collection are called [**Items**](https://docs.n8n.io/getting-started/key-concepts/#item).

<img src="./course_images/explanation_items.png">

An n8n node performs its action on each item of incoming data.

<img src="https://docs.n8n.io/assets/img/customer_datastore_node.7b7875b0.png">


## Creating data sets with the Function node

Now that you are familiar with the n8n data structure, you can use it to create your own data sets or simulate node outputs. To do this, use the *Function node* to write JavaScript code defining your array of objects with the following structure:

```javascript
return [
  {
    json: {
      key: 'value',
      }
    }
  ];
```

For example, the array of objects representing the Ninja turtles would look like this in the *Function node*:

<img src="./course_images/exercise_function_notNested.png">

!!! note "JSON objects"
		Notice that this array of objects contains an extra key: **`json`**. n8n expects you to wrap each object in an array in another object, with the key `json`.

<img src="./course_images/explanation_dataStructure.png">

It's good practice to pass the data in the right structure used by n8n. But don't worry if you forget to add the `json` key to an item, n8n (version 0.166.0 and above) adds it automatically.


You can also have nested pairs, for example if you want to define a primary and a secondary color. In this case, you need to further wrap the key-value pairs in curly braces `{}`.

!!! note "n8n data structure video"
	For a more detailed explanation of data structure in n8n, watch [this meetup talk](https://www.youtube.com/watch?v=mQHT3Unn4tY).

### Exercise
In a *Function node*, create an array of objects named `myContacts` that contains the properties `name` and `email`, and the `email` property is further split into `personal` and `work`.

??? example "Show me the solution"

		In the *Function node*, in the JavaScript Code field you have to write the following code:

		```
		var myContacts = [
			{
				json: {
					name: 'Alice',
					email: {
						personal: 'alice@home.com',
						work: 'alice@wonderland.org'
					},
				}
			},
			{
				json: {
					name: 'Bob',
					email: {
						personal: 'bob@mail.com',
						work: 'contact@thebuilder.com'
						},
				}
			},
		];

		return myContacts;
		```

		When you execute the Function node, the result should look like this:

		[image](./course_images/exercise_function.png)


## Referencing node data with the Function node

Just like you can use [expressions](https://docs.n8n.io/nodes/expressions.html) to reference data from other nodes, you can also use some [methods and variables](https://docs.n8n.io/nodes/expressions.html#method-evaluateexpression-expression-string-itemindex-number) in the *Function node*.

### Exercise
Let's build on the previous exercise, in which you used the *Function node* to create a data set of two contacts with their names and emails. Now, connect a second *Function node* to the first one. In the new node, write code to create a new column named `workEmail` that references the work email of the first contact.

??? example "Show me the solution"

    In the Function node, in the JavaScript Code field you have to write the following code:

    ```javascript
    items[0].json.workEmail = items[0].json.email['work'];

    return items;
    ```

    When you execute the Function node, the result should look like this:

    <img src="./course_images/exercise_function_reference.png">



## Transforming data

The incoming data from some nodes may have a different data structure than the one used in n8n. In this case, you need to [transform the data](https://docs.n8n.io/getting-started/key-concepts/transforming-data.html#transforming-data), so that each item can be processed individually.

The two most common operations for data transformation are:
- Creating multiple items from one item
- Creating a single item from multiple items

There are several ways to transform data for the purposes mentioned above:

- With the *HTTP Request node*, you can toggle the option `Split Into Items` to create multiple items from a single item. This is the easiest way to transform incoming web data with one click.
- With the [*Item Lists node*](https://docs.n8n.io/nodes/n8n-nodes-base.itemLists), you can `Split Out Items` or `Aggregate Items`. This node is the easy way to modify the structure of incoming data that contain lists (arrays), without needing to use JavaScript code in the *Function node*.
- With the *Function node*, you can write [JavaScript functions to modify the data structure](https://docs.n8n.io/reference/javascript-code-snippets.html#modify-data-structure) of incoming data.

    - To create multiple items from a single item, you can use this JavaScript code:

			```javascript
			return items[0].json.map(item => {
				return {
					json: item
				}
			});
			```

    - To create a single item from multiple items, you can use this JavaScript code:

      ```javascript
      return [
        {
          json: {
            data_object: items.map(item => item.json)
          }
        }
      ];
      ```

### Exercise

Use the *HTTP Request node* to make a GET request to the Poemist API `https://www.poemist.com/api/v1/randompoems`. Transform the incoming data with the `Split Into Items` option and [with the *Function node*](https://docs.n8n.io/reference/javascript-code-snippets.html#_1-create-multiple-items-from-a-single-item).

??? example "Show me the solution"

		To get the poems from the Poemist API, execute the *HTTP Request node* with the following parameters:
		- Authentication: None
		- Request Method: GET
		- URL: https://www.poemist.com/api/v1/randompoems

		The result should look like this:

		<img src="./course_images/exercise_transforming_noSplitItems.png">

		To transform the data from the *HTTP Request node*, toggle the switch `Split Into Items`. The result should look like this:

		<img src="./course_images/exercise_transforming_splitItems.png">

		To transform the data with the *Function node*, connect this node to the *HTTP Request node* (without the toggle for splitting data) and write the following code in the JavaScript Code field:

		```javascript
		return items[0].json.map(item => {
			return {
				json: item
			}
		});
		```

		The result should look like this:

		<img src="./course_images/exercise_transforming_function.png">

