`# Type Design
Some various design approaches for the media type design

## Custom JSON representation design
This concept is inspired by the Neo4j API

```
{
	<!-- There's an issue here in that in the current model, the semantics are different between a collection representation and an item representation -->
	"bugs":[
		{
			"data" : {
				"title" : "Bug 1",
				"description" : "This is an example of a bug description",
			},
			"assigned_to" : "http://host/26119E00-0ADD-46D7-8580-47475757E07E",
			"backlog" : "http://host/bugs/backlog",
			"working" : "http://host/bugs/working",
			"staging" : "http://host/bugs/staging",
			"qa" : "http://host/bugs/qa",
			"done" : "http://host/bugs/done",
			<!-- NOTE: treat link keys similar to HTML class attribute in that multiple values are space-delimited -->
			"move working" : {
				"href" : "http://host/bugs/working",
				"method" : "POST",
				"fields" : {
					<!-- TODO: is this overkill or can/should this information be presented out of band? 

					each field here would follow the attributes and possible values presented in the HTML5 specification (http://www.w3.org/TR/html5/single-page.html#the-input-element)
					-->
					"id" : { "type" : "hidden", "value" : 42 },
					"comments" : { "type" : "text" }
				}
			},
			"move backlog" : {
				"href" : "http://host/bugs/backlog",
				"method" : "POST",
				"fields" : {
					"id" : { "type" : "hidden", "value" : 42 },
					"comments" : { "type" : "text" }
				}
			},
			"move done" : {
				"href" : "http://host/bugs/done",
				"method" : "POST",
				"fields" : {
					"id" : { "type" : "hidden", "value" : 42 },
					"comments" : { "type" : "text" }
				}
			}
		}
	]
}
```

## Custom JSON representation design without embedded links
Trying to add more symmetry to the design

### big ideas 
* Every representation contains one or more JSON records
* A JSON record has a key that is a URI
* A URI is absolute and can be remote or local (e.g. '#')
* A JSON record has a value that is an object
* A JSON record value object contains a set of links and a data object
* A link can be 
   * A string value (outbound or templated) 
   * An object containing prescription for idempotent/non-idempotent
   * An array for a collection of links of the same type
      * Prev and next links are simply other link types (e.g. keys) in the array
* A data object simply has a key of "data" with the value being an object of key-value pairs

```
{
	"http://host/bugs/backlog" : {},
	"http://host/bugs/backlog#b1" : {},
	"http://host/bugs/backlog#b2" : {},
	"http://host/bugs/backlog#b3" : {},
	"http://host/bugs/backlog#b4" : {}
}
```

Expanding on this a bit

{
	"http://host/bugs/backlog" : {
		"data" : [
			"http://host/bugs/backlog#b1",
			"http://host/bugs/backlog#b2",
			"http://host/bugs/backlog#b3",
			"http://host/bugs/backlog#b4",
		],
		"self" : "http://host/bugs/backlog",
		"backlog" : "http://host/bugs/backlog",
		"working" : "http://host/bugs/working",
		"staging" : "http://host/bugs/staging",
		"qa" : "http://host/bugs/qa",
		"done" : "http://host/bugs/done"
	},
	"http://host/bugs/backlog#b1" : {
		"data" : {
			"title" : "Bug 1",
			"description" : "This is an example of a bug description",
		},
		"self" : "http://host/bugs/b1",
		"assigned_to" : "http://host/26119E00-0ADD-46D7-8580-47475757E07E",
		<!-- ...  -->
		"icebox": "http://host/bugs#icebox"
	}
	<!-- ...  -->


	<!-- Example scenario from Shy -->
	"bugs" : [
		"http://host/bugs/backlog#b1",
		"http://host/bugs/backlog#b2",
		"http://host/bugs/backlog#b3",
		"http://host/bugs/backlog#b4",
	]

	"http://host/bugs#icebox": [
		"host#b8",
		"host#b9",
		"host#b10"
	]
}

## Siren JSON representation design
This concept uses the application/vnd.siren+json media type to represent bugs data.

This kind of JSON metadata type seems a little bit overkill considering that:
1) there's no natural user agent for the type (at least none that I'm aware of)
2) I would still need to build on top of this type in order to apply domain semantics

```
{
	"class" : ["bugs"],
	"entities" : [
		{
			"class" : ["bug"],
			"rel" : [],

			TODO: add actions to move to various states

		}
	],
	"actions" : [
		"name" : "add-item",
		"title" : "Add Item",
		"method" : "POST",
		"href" : "",
		"type" : "application/x-www-form-urlencoded"   <!-- NOTE: for this, you would want to be able to pull in different modules based on the value of this field -->
		"field" : [
			{ "name" : "title", "type" : "text"}
		]
	],
	"links" : [
		{ "rel" : ["self"], "href" : "" },
		{ "rel" : ["backlog"], "href" : ""},
		{ "rel" : ["working"], "href" : ""},
		{ "rel" : ["qa"], "href" : ""},
		{ "rel" : ["done"], "href" : ""},
	]
}
```

# Notes

* if adopting a media type such as siren that my representation design builds on top of, is there a generic siren procesor (or if not, I could write one) that I could use to effectively get the equivalent of a DOM that I could work with rather than dealing with the raw representation? 
   * if this held, is there some kind of selector grammar that could build on top of a DOM?
   * could I provide functions related to objects - e.g. action[n].submit()? Another example would be that I could provide some level of validation.
   * need to take a look at the siren npm package to see how much of this may already be implemented

# TODO
* Show UX for forms (both item forms and response forms)
* generate item JSON from ??
* geneate state list from ??
* write the HTTP endpoint to capture update events and generate items

# Process

* The goal for this type of process is that we let the client drive the design of the resource model solely through the lens of the representations.
* As such, there should be little or no friction for standing up a server for basic resources so that we can start building out the client
* The first step, then is to create a directory for resources and start creating a bunch of different files (in this case, we'll create JSON files)
* We then need to stand up a lightweight server to host those files. We don't need anything production quality here, because the ultimate goal is to be able to push these resources into some kind of cloud storage (e.g. blob storage) - and we want to be able to continue to generate the represenations as static resources rather than converting this to some kind of dynamic Web API
* Python has a really simple HTTP server for doing exactly this. Simply run ```python -m SimpleHTTPServer``` and it will start a Web server, rooted in the directory that you specify.
