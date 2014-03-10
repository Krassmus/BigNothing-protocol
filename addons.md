# 6. Modules and Add-Ons

BigNothing is only a protocol. Thus there can be and hopefully will be many implementations that might be different in a lot of ways.
In the section *activities* we learned that the contents of a group (even the virtual public group) can be of any type and don't need to be restricted to postings and comments. But how do we display other content-types to the user? For example, if one user is editing a wiki page on his/her server, how do users of another server see the same wiki-page and - even more difficult - in the same way the first user wanted it to be displayed?

We need to define a small, simple, mighty and secure way for add-ons in our network that work on every server. There are two ways to achieve that. The first one might be to define a virtual new language (let's call it BigNothingProgrammingLanguage BNPL), to demand from any implementation of the protocol to be able to interpret this new language, run it in a sandbox and let the language or the code display the content to the user). And the second way is to use webstandards to achieve the same thing. Of course we take the webstandards and screw everything with the new language-stuff.

## Structure of an Add-On

The Add-On MUST BE identified in an activity by an IRI, an URL for a description and an URL for a javascript-file. This could look like that:

    {
        "actor": "<IRI of an identity>",
        "verb": "post",
        "object": {
            "objectType": "comment",
            "id": "123751",
            "iri": "http://mydomain/activities/123751",
            "content": "bla bla bla",
            "published": "2011-02-10T15:04:55Z",
            "encrypted": 1,
            "stream-module": {
                "iri": "<IRI for the module>",
                "description-url": "<URL to a JSON-file which describes what the module is doing>",
                "module-url": "<a URL to a ZIP- or JS-file>"
            }
        },
        "published": "2011-02-10T15:04:55Z",
        "target": "<IRI of a group or 'public' instead>",
        "signature": "<the identity signed the JSON-string of the object here>"
    }

So you see we have added a `stream-module` to the attributes of the content. The IRI is the international global ID of the module. The `description-url` is a URL to a JSON-file that looks like this:

    {
        "iri": "<IRI of the module>"
        "name": "<name of the module>",
        "avatar": "<some logo of the module>", //optional
        "description": "<small description, not longer than 100 letters>", //optional
        "module-url": "<URL to the module itself represented by a ZIP-file or a JS-file>",
        "large-description-url": "<URL to another page where the module can be described more precisely>", //optional
        "service-access": {
            //the indexes are service-names the API can provide 
            //and the values are the words "mandatory" for services the module absolutely depends to
            //and "optional" for a nice-to-have service, but the user may decline access to the service
            "stream-renderer": "mandatory",
            "group-page": "mandatory",
            "contact-access": "optional",
            ....
        }
    }

The first few points are straigh-forward. The `iri` is again the IRI-identifier of the module. The `avatar` and `description` give more information to the user about the module. The `module-url` is a repetition of the module-url given in the content's `stream-module`. The `large-description-url` is a simple external link to a description on a webpage, where the provider of the module can display some more information like screenshots and so on.

The `service-access` is an object that defines what interfaces the module relies on and what it wishes to have access to. Some might be needed by module to work and some might be wished by the module to give the user full usability. A module without any `service-access`-entries is useless and will do nothing for the user.

* **stream-renderer**: will render some activities in the user's stream. Now imagine this activity to look slightly different than a casual posting. It could be a poll with a question and several possible answers. Or it could be advertising, a video, a schedule with possibility to add the schedule to the calendar and so on.
* **group-page**: an own page in a group that can contain almost any kind of information and interactivity. Imagine it as a wiki-module in a group or as a group-chatroom, as a filesystem, a picture-galery, elearning-modules or anything else.
* **contact-access**: the module demands access to the contacts of the user to be more social itself. A game could connect players with each other, when it knows that the players are contacts in the BigNothing network for example.

## Activating the module

Each person needs to activate the module before the module can access the demanded services. This is to make sure that the user is not spammed by the module/add-on without wanting it.

A small message appears to the user that there is a module that wants to have access to the needed services. For the optional services the user can also select checkboxes to tell the BigNothing-implementation that the services should be enabled. This information is only saved for the login and not for the identity the user is using right now. So this information does not need to be sent to any other server different to the server of the login.

The settings for a module are saved and can be altered at another time.

## The module worker

Once a module is activated the server downloads the `module-url`. If it is a ZIP-file, the server unzips it. If it is a .js-file the server stores it at the same place the ZIP-file would be unzipped to. All other types are declined by the server and won't be downloaded or stored.

Now there should be at least a file for the module that is named `worker.js`. This file MUST BE a [shared webworker](http://www.w3.org/TR/workers/#shared-workers-and-the-sharedworker-interface) programmed in javascript. It will be invoked by the browserscript of the user. This shared worker is in its own sandbox. So neither the browser script can change the worker nor the worker can make any changes to the browser-script that are not part of our protocol. 

For each service that has been activated by the module and the user, the browserscript sends messages to the the worker and receives messages from the worker that tell the browserscript how to - for example - display a special posting like a poll in the user's stream or anything else.

The only the browser-script and the shared worker can communicate with each other is the [web messaging protocol](http://www.w3.org/TR/webmessaging/). Those messages consist of a data variable that can be any javascript-structure you want. For all services for BigNothing modules the data must have such a form:

    {
        "service": "stream-renderer",
        "input-data": { // this could be anything
            "id": "#posting_1234",
            "order": "render",
            "rawcontent": "{\"type\":\"poll\",\"question\":\"Is that cool?\",\"answers\":[\"yes\",\"no\"]}"
        },
        "output-data": { //data sent back from the worker to the browser-script
            "htmlcontent": "<form><h2>Is that cool?</h2><ul><li><label><input type=radio name=\"cool\" value=\"yes\"">yes</label></li><li><label><input type=radio name=\"cool\" value=\"no\">no</label></li></ul></form>"
        }
    }

The browser-script knows about the services the module-worker can talk to. So in this case the service is `stream-renderer` and if an activity appears in the stream that should be rendered by the worker (defined in the activity's content attribute `stream-module`, see above) the browser-script posts a message to the shared worker like above but without the `output-data` attribute. The `output-data` is to be filled by the shared worker, when it returns its answer to the browserscript.

So the shared worker received the message, inspects the `input-data` and calculates the `output-data`. In this case the output data is simple HTML. But the worker has all the time in the world to send AJAX-requests, handle websockets or even calculate huge prime-numbers. Shared worker are handled by the browser in a different process so they can't harm the user-experience of the user's browser-tab.

So the browser-script receives the data above from the worker and can handle it. The `input-data` MUST BE still there, so that the browser-script knows what outgoing message the worker was processing (there could be multiple messages in the pipe, because the worker works asynchronously) and can use the `output-data` to display the activity correctly. What is the `input-data`, what does the browser-script expect as the `output-data`? This is all defined by the service. So each available service might have different specification for the workflow and the data-formats.