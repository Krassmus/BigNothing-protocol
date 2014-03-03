# 3. Authentication and verifying

For authentication we only have one simple mechanismus in BigNothing: asynchronous RSA. We use it for encryption of data to make sure that only the humans we want to see the contents are able to see it. And we use it for signing the data-package so that everyone can be sure that the content is coming from the correct identity.

If an activity belongs to a non-public group, it has to be encrypted:

	{
		"actor": {
			"objectType": "identity",
			"<IRI of the identity>"
		},
		"verb": "post",
		"object": {
			"id": "<some ID or IRI of the object>",
			"objectType": "posting",
			"content": "<text encrypted with the private key of the group>",
			"published": "2014-03-03T21:20:50.52Z"
		},
		"signature": "<RSA signature of the JSON-representation of the pure object, signed by the actor identity>"
	}

The content itself is encrypted so only the persons that are member of the group can receive the private key of the group and only they can decrypt the content. And because there is a signature of the JSON-serialized object of the activity (with all attributes like published and the encrypted contents) the server containing the group can receive the activity as a whole string from any source in the world. And as long as the signature is correct (and the identity is member of the group), the activity is verified and can be inserted into the group's contents.

What does that mean practically? It means that federating contents can take different ways. Of course there is the way of a user writing a posting in the website he/she is seeing, this posting is encrypted, delivered to his/her server, that server delivers that posting to the server hosting the group and that server is verifying the activity and inserting it. But the user could also have the information of the endpoint of the groupserver in his/her browser, write something and push it directly to the server hosting the group, which is then verifying the activity and inserting it. But it might even get more fancy. One user could start a group video-text-chat via WebRTC, that user gathers all activities from all other participants via peer2peer-mechanismus and - if desired - pushes the whole conversation to the server hosting the group, which is then verifying the possible 200 text-messages at once and inserting them.

So it does not matter which browser or which desktop-program or which command-line-script is pushing the activities to the hostserver - as long as the signature is correct, the activity will get verified and inserted to the group's contents. No OAuth is needed here, no salmon protocol or stuff like that. And the benefit of this mechanism is clear: we are flexible in distributing the contents in the network. Still XMPP-chats or WebRTC is possible, and - maybe the most important thing - it is fast and can reduce bandwidth of the servers.