# Migrating from Embedding JSAPI v1 or v2 to Embedding API v3 (Migration Guide)

> The initial release of the Embedding API library (version 3.0) was made available as part of the Tableau 2021.4 launch. To find out about using the current version of the library (now 3.4), see the documentation for the [Tableau Embedding API v3 documentation](https://help.tableau.com/current/api/embedding_api/en-us/index.html).



## Introduction

With Tableau’s Embedding API v3, we have redesigned the experience for embedding to make your life easier, to modernize the experience, and to enable new functionality.
(If you are unfamiliar with the JavaScript API v2, you can [read its documentation](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api.htm).)
Some parts of using the API are the same as the JavaScript API v2, and others have just slightly different syntax. But the biggest difference is the initialization of the viz, which uses web components.
Web components are a standard available in all modern browsers, that allow you to add custom, and 3rd party, functionality to your web page using html syntax as easily as adding an image, link, or div. Using the web component technology, we have created a tableau-viz component that you can use to add visualizations to your web pages.

## How to use this guide

This guide has a few goals:
1) To help you understand key concepts of using the Embedding API v3 
2) to solicit feedback from our community on changes we intend to make 
3) to help you migrate existing content to v3.

If you have comments or feedback, please submit an [Issue](https://github.com/tableau/embedding-api-v3-guide/issues) on this Github repository.

## Roadmap

For information about the next release of the Tableau Embedding API (version 3.6), see the [Tableau Embedding API v3 Developer Preview
](https://embedding.tableauusercontent.com/preview/getting-started-v3.html). The Developer preview includes new methods for programmatically exporting views directly from Tableau without requiring user intervention, support for custom error handling for connected apps, and create embedded workbooks from scratch, with or without a data source.

### Strategies for adopting the Embedding API v3

* Anywhere you are using the previous versions’ copy-paste embed code, we recommend replacing with the new copy-paste embed code which, as you will see below, uses the Embedding API v3.
* If you are starting a new embedding project, you should use the Embedding API v3.
* For existing JavaScript API v2 projects, we recommend migrating your code as your timeline allows, because the Embedding API v3 can enable more robust embedding and because we will eventually end support for the JavaScript API v2.

## Getting Started (or replacing ‘Embed Code’ from previous versions)

In previous versions of Tableau Server, there existed a JavaScript API v1. It is almost exclusively used in the ‘embed code’ from Tableau Server’s Share dialog.

```html
<script type='text/javascript' src='https://myserver/javascripts/api/viz_v1.js'></script>
<div class='tableauPlaceholder' style='width: 1500px; height: 827px;'>
    <object class='tableauViz' width='1500' height='827' style='display:none;'>
    <param name='host_url' value='https%3A%2F%2Fmyserver' /> 
    <param name='embed_code_version' value='3' />
    <param name='site_root' value='&#47;t&#47;Superstore' />
    <param name='name' value='MyView' />
    <param name='tabs' value='no' /><param name='toolbar' value='yes' />
    <param name='showAppBanner' value='false' /></object>
</div>

```

To migrate from the JavaScript API v1 to the new JavaScript API v3, you can copy the new embed code which will give you something like:

```html
<script src = "https://myserver/javascripts/api/tableau.embedding.3.0.0-alpha.23.js"></script>
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view" 
    device="phone" toolbar="bottom" hide-tabs>
</tableau-viz>
```

*Check the documentation for the current version of the .js file.*

You will notice that the syntax of v3 is just as easy to copy-paste into your html and appears simpler because of the <tableau-viz> component and its ability to customize the properties of the embedded viz without nesting <param> elements.
Another key benefit to v3 is how easy it is to go from this simple copy-paste embed scenario to one where you interact with the viz via API. You can get started with the API simply by pasting in the v3 version of the embed code and then start layering on interactions and listeners as you will see below.

Before we do that, let’s take a closer look at the key parts of initializing an embedded viz.

## Access the API

Accessing the API happens similarly to v1 and v2: You simply include the correct .js file, this time including the tableau-3.js (or a variation of it) from your Server. As before, you can reference the .js from your on-premise Server, from Tableau Online, or from Tableau Public.

## Initialize the API

In the JavaScript API v2, you initialize the viz by adding an empty div object to your html and then create a new tableau.Viz object referencing the div and the viz url. Example:
 
HTML (v2 - will also work in v3)

```html
<div id='tableauViz'></div>
```

JavaScript (v2 - going away. For v3 JS initialization see [Alternative Approach: Initialization via JavaScript](#alternative-approach-initialization-via-javascript))

```javascript
let placeholderDiv = document.getElementById("tableauViz");
let src = "http://my-server/views/my-workbook/my-view";
let options = {}
let viz = new tableau.Viz(placeholderDiv, url, options);
```

In the Embedding API v3, the initialization step is simplified and can happen all inside of your html, thanks to the new tableau-viz web component:

HTML:

```html
<tableau-viz id="tableauViz" 
      src='http://my-server/views/my-workbook/my-view'>
</tableau-viz>
```

### Configuration:

To specify options on how to initialize the Viz in the JSAPI v2, you would add those to the options object that is included in the Viz object’s constructor’s arguments. In the Embedding API v3, you can simply add those as properties of the viz component:


```html
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view" 
    device="phone" toolbar="bottom" hide-tabs>
</tableau-viz>
```

Note: for properties that accept a variety of values, such as src, device, and toolbar the syntax is `<property>=“<value>”`. For properties which are boolean, you either include the flag, to effectively set it to true (as shown with hideTabs), or omit it, to effectively set it to false.

Here is the list of properties you can add to your viz object:

|HTML Property	|JS Property	|Accepted Values	|Description	|
|---	|---	|---	|---	|
|hide-tabs	|Viz.hideTabs	|boolean	|Indicates whether tabs are hidden or shown.	|
|toolbar	|Viz.toolbar	|"top", "bottom", "hidden"	|Indicates the position of the toolbar or if it should be hidden	|
|height	|Viz.height	|\<string\> (e.g. "800px" or "800")	|Can be any valid CSS size specifier. If not specified, defaults to the published height of the view.	|
|width	|Viz.width	|\<string\> (e.g. "800px" or "800")	|Can be any valid CSS size specifier. If not specified, defaults to the published height of the view.	|
|device	|Viz.device	|\<string\>	|Specifies a device layout for a dashboard, if it exists. Values can be `default`, `desktop`, `tablet`, or `phone`. If not specified, defaults to loading a layout based on the smallest dimension of the hosting `iframe` element.	|
|instance-id-to-clone	|Viz.instanceIdToClone	|\<string\>	|Specifies the ID of an existing instance to make a copy (clone) of. This is useful if the user wants to continue analysis of an existing visualization without losing the state of the original. If the ID does not refer to an existing visualization, the cloned version is derived from the original visualization.	|
|disable-url-actions	|Viz.disableUrlActions	|boolean	|Indicates whether to suppress the execution of URL actions. This option does not prevent the URL action event from being raised. You can use this option to change what happens when a URL action occurs. If set to `true`and you create an event listener for the `URL_ACTION` event, you can use an event listener handler to customize the actions. For example, you can direct the URL to appear in an `iframe`on your web page. See [URL Action Example](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#urlaction).	|

*When adding properties to the tableau-viz component, use the HTML Property name. When [configuring via JavaScript (as explained below)](#alternative-approach-initialization-via-javascript), use the JS Property.*

### Filtering during initialization

In JSAPI v2 you can specify filtering to occur, by specifying field-value in the url or options object. In the Embedding API v3, you can accomplish the same thing by add <viz-filter> elements as children to your <tableau-viz> objects. You can also set an initial parameter value.

```html
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view" 
    device="phone" toolbar="bottom" hide-tabs>
    <viz-filter field="country" value="USA" />
    <viz-filter field="state" value="California,Texas,Washington"></viz-filter>
    <viz-parameter name="currency" value="dollars" />
    <viz-range-filter field="profit" min="0" max="10000"></viz-range-filter>
    <viz-relative-date-filter field="Date" anchor-date="March 17, 2001"
        period-type = "Year" range-type = "LastN" range-n = "2"></viz-relative-date-filter>
</tableau-viz>
```

*For viz-range-filter and viz-relative-date-filter, use the properties from* [*RangeFilterOptions*](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#rangefilteroptions_record) *and [RelativeDateFilterOptions](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#relativedatefilteroptions_record), but convert them to dash-casing.*

One important difference is that in the JSAPI v2 the filters that were loaded on initialization were added to the url of the viz. This limited the amount of filters that could be added, because url’s have a maximum length of 2048 characters. In the Embedding API v3, the filtering occurs separately from the viz url, but still occurs on initial load.


### Event listeners

You can also add event listeners to the <tableau-viz> object

```html
<tableau-viz id="tableauViz" 
    src="http://my-server/views/my-workbook/my-view"
    onMarksSelected="handleMarksSelection">
</tableau-viz>
```

(In the above example, handleMarksSelection() is a function which is specified in the JavaScript somewhere

## Alternative Approach: Initialization via JavaScript

There are some scenarios where you may prefer to configure and initialize the viz via JavaScript instead of the above HTML <tableau-viz> component approach. Here is an example that demonstrates all of the above functionality using JavaScript.

HTML:

```html
<div id='tableauViz'></div>
```


JavaScript:

```javascript
import {TableauViz} from '../tableau.embedding.3.0.0-alpha.23.js'

let viz = new TableauViz();

viz.src = 'http://my-server/views/my-workbook/my-view';
viz.toolbar = 'hidden';
viz.addFilter('Region', 'Central');
viz.addEventListener('marksSelected', handleMarksSelection);

document.getElementById('tableauViz').appendChild(viz); 
```

Important notes about the above approach:

* `new TableauViz()` does not render the viz. It creates a viz object so that you can configure the viz before it renders. It renders when you add it to the DOM, through something like document.body.appendChild(viz);
* You can use JavaScript’s built-in `document` object to choose where in your html you want to add the viz. The above example uses an empty div and getElementById similar to the JavaScript API v2 approach, but you may other approaches depending on your scenario. [Read more about JavaScript’s document object.](https://www.w3schools.com/js/js_htmldom_document.asp)
* All of the [configuration options listed above as properties of the viz object](#configuration), can be defined in the same manner as `viz.toolbar` above.
* Changing the properties, as shown in this example, will re-render the viz if it is already rendered. This is especially important to note for filtering. The addFilter method is not intended to be used after the viz has been initialized, instead use applyFilterAsync (and the other Filtering methods when appropriate).



## Interacting with the Viz after initialization

All of the above discusses how to initialize the Viz and to configure it during initial load. You will likely want to interact with the Viz — filter, add/remove event listeners, change parameters, query data, etc. — after the Viz loads. The experience for accomplishing those things is very similar to the JavaScript API v2, except where noted below.

### Accessing the Viz object

If you [initialized the Viz viz JavaScript](#alternative-approach-initialization-via-javascript), then you already have a Viz object which you can interact with. For example:


```javascript
import {TableauViz} from '../tableau.embedding.3.0.0-alpha.23.js'

let viz = new TableauViz();

viz.src = 'http://my-server/views/my-workbook/my-view';
viz.toolbar = 'hidden';
viz.addFilter('Region', 'Central');
viz.addEventListener('marksSelected', handleMarksSelection);

document.getElementById('tableauViz').appendChild(viz);

// Later
let sheet = viz.workbook.activeSheet;
sheet.applyFilterAsync("Container", ["Boxes"], FilterUpdateType.Replace);
```

But if you created a <tableau-viz> object in your html, you need to use document.getElementById to access the viz in your JavaScript:


```javascript
let viz = document.getElementById('tableauViz');

// Later

let sheet = viz.activeSheet;
sheet.applyFilterAsync("Container", ["Boxes"], FilterUpdateType.Replace);
```

*Note: In both of the above examples, the Viz will not be ‘interactive’ immediately after you initialize the Viz object. If you plan on filtering or doing some interaction immediately after initialization you should use the onFirstInteractive event listener to ‘wait’ for the viz object to be ready. This was also true in the JavaScript API v2.*


### Accessing other Objects and Namespaces

The relationship between objects/namespaces in v3 remains the same [as in the JavaScript API v2](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#top_level_class_diagram). However, the syntax follows modern JavaScript practices and treats related objects as properties instead of returning them in getter methods.
For example, to access the activeSheet’s worksheets in the JavaScript API v2, you would previously use 

```javascript
viz.getActiveSheet().getWorksheets();
```

In v3, you use

```javascript
viz.activeSheet.worksheets;
```

The only exception to this is when calling an asynchronous method, such as getUnderlyingDataAsync() which keeps the same syntax as in v2. The reference will list the appropriate properties and methods for each Namespace.

In v2, many methods returned a collection of objects instead of a native array, allowing you to call .get to find the individual object that you wanted to act upon. For example:

```javascript
let sameSheet = workbook.getPublishedSheetsInfo().get("Sheet 1");
```

In v3, these properties return native JavaScript arrays and you can use JavaScript’s .find to access the individual object:

```javascript
let sameSheet = workbook.publishedSheetsInfo()
    .find(sheet => sheet.name == "Sheet 1");
```

### Querying Data

In v2, getUnderlyingDataAsync() was deprecated and only available in Tableau 2020.1 or earlier. Therefore, it is not available in v3. Instead [use getUnderlyingTablesAsync() and getUnderlyingTableDataAsync()](https://help.tableau.com/current/api/js_api/en-us/JavaScriptAPI/js_api_ref.htm#gettabledata).

getSummaryDataAsync() remains in v3 with the same syntax. However, the following options have been added to getSummaryDataOptions.

|Option	|Accepts	|Description	|
|---	|---	|---	|
|columnsToIncludeById	|int[] (columnID)	|Allows you to specify the list of columnIDs to return	|
|maxRows	|int	|Returns the first set of rows up to the number specified	|
|onlyFormattedValues	|bool	|Only return the formatted values (as specified by the viz author) and not the native values	|
|onlyNativeValues	|bool	|Only return the native values and not the formatted values	|

There are also two new supporting additions:

* getSummaryColumnsInfoAsync(): returns only the Column info for when you just need to retrieve metadata about the columns.
* fieldID property of Column object: for use in columnsToInclude.

### Resetting the active custom view

In v2, to reset the active custom view to the original view, you would use `workbook.showCustomViewAsync()` without providing any arguments.

In v3, to reset the active custom view to the original view, set `viz.src` to its original value used to load the viz.

## Things that remain the same

### Filtering, Selecting Marks, and Changing Parameters

Filtering, selecting marks, and changing parameters after initialization remain the same:

```javascript
worksheet.applyFilterAsync("Product Type", "Coffee", 
    FilterUpdateType.Replace);

```

```javascript
workbook.changeParameterValueAsync("Product Type", "Coffee");

```

*changeParameterValueAsync is available from Tableau Server 2022.3, Use tableau.embedding.3.3.0.js version for this functionality.*

```javascript
worksheet.selectMarksByValueAsync("Product", ["Caffe Latte"], 
    SelectionUpdateType.Replace);

```

### Event Listeners

Adding and removing event listeners after initialization remains the same:

```javascript
viz.addEventListener("marksSelection", function (marks) {
   changeMySelectionUI(marks);
});
```

```javascript
viz.removeEventListener("marksSelection", changeMySelectionUI);
```
