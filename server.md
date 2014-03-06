# 2. Server discovery

The first resource accessable via webfinger is the server itself. It provides information about the server, access-endpoints-points and some statistics. The resource IRI is `http://yourserver/discovery` and the retrieved properties look like this:

	{
		"domain": "<the prefered domain or ip with port>",
		"ssl": 1 //or 0 if it doesn't support ssl,
		"activity-endpoint": "<one URL that can be accessed via POST and receives activities to be inserted into the database>",
		"number-of-identities": "123", // optional: just a number for statistics
		"number-of-groups": "12", // optional: just a number for statistics
		"search-groups": "<URL for a GET request to retrieve all known group for a filter>", //optional
		"search-identities": "<URL for a GET request to retrieve all known (and searchable) users for a given filter>" //optional
	}

Most important is the activity-endpoint here and if the server supports SSL or not. Servers of the BigNothing network SHOULD support SSL, but they don't need to. 

## Searches

To be defined, but since it's an optional flag, it's not so important right now.