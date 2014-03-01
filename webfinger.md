# Webfinger in BigNothing

In BigNothing we want to use the [webfinger protocol in its JSON-variant](http://tools.ietf.org/html/rfc7033) to provide information about ressources and end-points.

For those who don't want to read RFCs, here's some breaking down for you:

* Your server MUST BE a webserver reachable over http and possibly over https.
* Your server MUST HAVE one very special adress-uri that is as follows: `http://yourserversadress/.well-known/webfinger`
* This resource MUST BE a dynamic script (reachable by HTTP-GET) that takes the parameter `resource` and the value of `resource` is most likely an identifier of an *identity* like `rasmus@blubber.it` or it is identified by its URI like `http://yourserveradress/users/Rasmus`.

When we say *identity*, we mean he item of the network.

Now the response of webfinger is always a JSON-object and so it's well defined to be an UTF8-encoded string. It looks like this:

    {
    	"subject": "URI adress of the entity, this is considered to be the ID",
    	"aliases": [
    		"<URI of this resource>",
    		"<optional email-like identifier rasmus@blubber.it - for identities only>"
    	],
    	"properties": {...},
    	"links": [...]
    }

The properties of the item are class-specific, which means different entities have different properties.