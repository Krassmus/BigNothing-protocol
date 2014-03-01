# Logins, identities, groups and postings

In BigNothing we suppose to have a very special scheme for items in our network. This scheme is fixed so that we can rely on other BigNothing-servers to have the same logic behind those entities.

To make it more precise a user in BigNothing is one entity, but a set of two classes of entities: one login and one or many identities. So we will try not to talk about users anyway, but of logins and identities.

## Logins

When you are a user as a human person and want to login to your server you need a login. That is a combination of a loginname, a password and some RSA-ringpair. Your login doesn't have a name or a profile pic. This is due to your identity and we separate between the login and an identity to provide you the possibility to have more than one identitiy or to share an identity with some other human persons. For example an identity could be a company and many people (logins) have the right to speak as the company-account.

In fact a login never does something in the network. A login without a connected identity is useless and cannot set any activities.

A login does not have an URI and cannot be accessed via webfinger as a resource of the server. It is mostly used internally in the server and only the public keys are occasionally sent to other servers to connect to identities.

## Identities

The identity is the main actor in the network. It is the subject of most activities. It can be connected with other identities (via friendships or followships), it must be connected to at least one login. But it is important to know that an identity is not equal with a login. In fact an identity can be on a different server than the login that is using the identity.

An identity is reachable via webfinger over the URI: `http://yourserver/identitiy/:id` .

And it has the following properties: 

	{
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

Now mind that the human person seems to need to enter a loginname and a password and a passphrase. This is designed to make end-to-end encryption possible. But in fact the server-software could also take care about the private-key of the login without letting the human person to even know about its existence. So you could implement the protocol by providing end-to-end encryption and not providing it. The protocol does not tell you which way you should implement it and it does not say which way is better. Both might be useful for different implementations. But this also means that if you are a human person and you want to use the BigNothing network for its easy end-to-end encryption you need to know how your contacts are using the network. It is as always: the network is only as safe as the implementation that is using the network.