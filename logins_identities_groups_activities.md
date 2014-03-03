# Logins, identities, groups and activities

In BigNothing we suppose to have a very special scheme for items in our network. This scheme is fixed so that we can rely on other BigNothing-servers to have the same logic behind those entities.

To make it more precise a user in BigNothing is one entity, but a set of two classes of entities: one login and one or many identities. So we will try not to talk about users anyway, but of logins and identities.

In an overview we want to discuss the following classes of entities:

![logins_identities_groups_activities](https://raw.github.com/Krassmus/BigNothing-protocol/master/assets/logins_identities_groups_activities.svg)

## Logins

When you are a user as a human person and want to login to your server you need a login. That is a combination of a loginname, a password and some RSA-ringpair. Your login doesn't have a name or a profile pic. This is due to your identity and we separate between the login and an identity to provide you the possibility to have more than one identitiy or to share an identity with some other human persons. For example an identity could be a company and many people (logins) have the right to speak as the company-account.

In fact a login never does something in the network. A login without a connected identity is useless and cannot set any activities.

A login can be accessed as a resource via webfinger with the [IRI](http://tools.ietf.org/html/rfc3987) `http://yourserver/logins/:loginname`. Remember that this [IRI](http://tools.ietf.org/html/rfc3987) is not necessarily the URI for the login and the login does not even necessarily have an URI, which means having a real webadress. The IRI looks like a webadress but is only used for identifying the login.

This is only done for inviting other logins to share an identity. So we only need to access the login-resource for the purpose of identitied which have multiple logins. This is how the properties of the resource look like.

	{
		"iri": "<the IRI of this login>",
		"public-key": "<base64encoded string of the public key>",
		"private-key": "<base64encoded string of an AES-encoded private key"
		"loginname": "<loginname>"
	}


## Identities

The identity is the main actor in the network. It is the subject of most activities. It can be connected with other identities (via friendships or followships), it must be connected to at least one login. But it is important to know that an identity is not equal with a login. In fact an identity can be on a different server than the login that is using the identity.

An identity is reachable via webfinger with the following [IRI](http://tools.ietf.org/html/rfc3987) `http://yourserver/identities/:id` or the email-like identifier `rasmus@yourserver.com`. And it has the following properties: 

	{
		"iri": "<the IRI of the identity>",
		"public-key": "<base64encoded string of the public key>",
		"private-key": "<base64encoded string of an AES-encoded private key",
		"logins": [ //users to access this identity
			{
				"login": "Rasmus",
				"server": "<domain or ip with optional port>",
				"public-key": "<base64encoded string of the public key>",
				"encrypted-passphrase": "<passphrase to identity.private-key encrypted for just this login>"
			},
			...
		],
		"name": "<some name>",
		"avatar": "<URL of a picture resource>"
	}
	
### Logging in as a person

As you see the identity has one private key that is AES-encrypted and each connected login has one attribute `encrypted-passphrase` which contains the passphrase to decrypt the private-key of the identity. So when a human person logs in, 

1. he/she enters the loginname and password, 
2. returns back the private key of the login (see there the ringpair), 
3. the human person can now decrypt the private key of the login with a first passphrase the human person has to know.
4. Now the server delivers all the identities connected to that login to the human person,
5. that human can now use his/her private-key to decrypt the `encrypted-passphrase` to receive the real passphrase,
6. and with this real passphrase the human can decrypt with AES the private key of the identity.

And voilà, the human person has now the raw private-key and can entirely act with the identities connected to the login.

Now mind that the human person seems to need to enter a loginname and a password and a passphrase. This is designed to make end-to-end encryption possible. But in fact the server-software could also take care about the private-key of the login without letting the human person to even know about its existence. So you could implement the protocol by providing end-to-end encryption or by not providing it. The protocol does not tell you which way you should implement it and it does not say which way is better. Both might be useful for different implementations. But this also means that if you are a human person and you want to use the BigNothing network for its easy end-to-end encryption you need to know how your contacts are using the network. It is as always: the network is only as safe as the implementation and the users that are using the network. The goal of BigNothing is only to make it possible to use the network as safe as it can be. But it is not a must.

## Groups

Groups in BigNothing have some kind of a double meaning. At first they are used to define an audience for a posting. The posting is then only readable and writable by the audience. So the group is just a set of identities in the network. But a group can be extended so that it has more than one posting attached to it. Also it could have an avatar, a name and some other things that make a group look more like an entity in the network.

Of course a group can be accessed via webfinger with the IRI [IRI](http://tools.ietf.org/html/rfc3987) `http://yourserver/groups/:id`. And the properties are as followed:

	{
		"iri": "<the IRI of the group",
		"public-key": "<base64encoded string of the public key>",
		"private-key": "<base64encoded string of an AES-encoded private key",
		"members": [ //an array of identities
			{
				"iri": "<iri of the identity>",
				"public-key": "<base64encoded string of the public key of the identity>",
				"encrypted-passphrase": "<AES passphrase to the private key encrypted for this member identity>",
				"name": "Rasmus Fuhse",
				"avatar": "<URL of an image as a webresource>"
			},
			...
		],
		"admins": [
			"<IRI of a member",
			...
		],
		"name": "Rasmus supergroup", //optional
		"avatar": "<URL of an image resource>", //optional
		"expand_audience": 1, //or 0 if not
		"modules": [ ... ] //see later for description of modules in groups
		....
	}

The group is probably the most complex entity in the network, because it is designed as a very flexible container for all kinds of sets of users with a special semantic or logic.

But at the beginning we see that each group has again its own public and private key. All contents of the module MUST BE encrypted with the public key of the group and can only decrypted with the private key which is AES encrypted. An identity can decrypt the AES-passphrase with the private key of the identity and then decrypt the private key of the group. With this private key the identity can now decrypt all contents in the group, read the contents and of course write something in the group. But for writing to the context of the group only the private key of the identity is needed to sign the activity. But we discuss this later in detail.

Whenever a member is leaving the group or has left the group the group needs to get a new ringpair, so that the old member cannot access the (new) contents of the group anymore. This is a task that might take a lot of time. But luckily this should not happen so often. But for now it is important to know that the private and public key of the group can change. So you better beware when caching the keys in your browser or server implementation.

### "Public"

There is also the possible "public" group that does not exist as an entitity in the network and does not have any attributes. But it is the group public postings belong to. No notifications are sent to the members of this group. No modules exist here and there are no specific semantics attached to this group. The "public" group is just a virtual word for content that belongs to "everyone who is interested".

## Activities

Whenever we have contents within a group or made by an identity, these contents are activities as described by [activitystrea.ms](http://activitystrea.ms/) JSON variant 2.0 . This means all contents MUST BE describable as activities with an identity as the author of the activity and this activity is the transport-format to send contents from one server to another.

An activity is merely a triple of subject, verb and object. There are some extensions like an id, timestamps, a target and so on. All activities MUST HAVE a target and the most obvious target is a group (or even the virtual public group). Leaving the target away an activity can be described as a triple `(subject, verb, object)`. An activity could be editing a wiki-page or writing a posting, uploading a photo and so on. 

Activities are just a flexible format for some kind of contents. It does not tell which activities are supported and which aren't. In fact all activity-types can be supported but there are a few types our network relies on.

* **posting**: usage-triple is `(identity, "post", "posting")`. 
* **following**: usage-triple is `(identity, "follow", identity)`.
* **request membership**: usage-triple is `(identity, "requestmembership", group)`
