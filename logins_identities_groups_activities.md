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