# MetaverseAPI - User Related Operations

Requests that create and manage list of users.

- [Users](#Users) - Get information on other users
- [Friends](#Friends) - Get information on friends
- [Connections](#Connections) - Get and set connections
- [Administrative](#Administrative) - administrative account requests

# Users

## GET /api/v1/users

Returns a list of users.

A user can only see the other accounts they are 'connected' to so
this request returns a subset of all accounts depending on the account 
of the requestor. If the requestor is an administrator and the query
parameter 'asAdmin=true' is part of the request, all user are returned.

The GET request url can have queries added which controls the user's returned by
the request. These queries are:

- per_page: maximum number of entries to return
- page: the group of "per_page" to return. For instance, if there are 100 users and `per_page=10` and `page=2`, the request will return entries 11 thru 20.
- filter: select type of user. A comma separated list of "connections", "friends"
- status: status of user. A comma separated list of "online"
- search: TODO: figure this one out
- asAdmin=true: if requesting account is an administrator, return all users

So `GET /api/v1/users?per_page=10&filter=friends&status=online` will return the first 10 users
who are online friends.

The response body is an "applicaton/json" structure that contains an array of user information.


```
    {
        "status": "success",
        "data": {
            "users": [
                {
                    "username": username,
                    "accountid": stringAccountId,
                    "images": {
                        "hero": heroImageURL,
                        "thumbnail": thumbnailImageURL,
                        "tiny": tinyImageURL
                    },
                    "location": {
                        "node_id": stringSessionId,
                        "root": {
                            "domain": {
                                "id": string
                                "network_address": stringHostname,
                                "network_port": intPortNum,
                                "ice_server_address": stringHostname,
                                "name": name,
                            },
                            "name": placeName
                        },
                        "path": "/X,Y,Z/X,Y,Z,W",
                        "online": bool
                    }
                },
                ...
            ]
        }
    }
```

## GET /api/v1/user/profile

Returns a user's profile.
The profile returned is of the account that is making the request (token in "Authorization:" header).

Not much information is returned and this will probably expand in the future.

The response body is an "applicaton/json" structure that contains an array of user information.

```
    {
        "status": "success",
        "data": {
            "user": {
                "username": userName,
                "accountid": stringAccountId,
                "xmpp_password": stringDeprecatedPassword,
                "discourse_api_key": stringDeprecatedAPIKey,
                "wallet_id": stringWalletId
            }
        }
    }
```

---

# Friends

## GET /api/v1/user/friends

Return a list of friends of the requesting account.

The response body is an "applicaton/json" structure that contains an array of user information.

```
    {
        "status": "success",
        "data": {
            "friends": [
                username,
                username,
                ...
            ]
        }
    }
```

## POST /api/v1/user/friends

Set a user as a friend. The other use must already have a "connection" with this user.


```
    {
        "username": stringUsername
    }
```

```
    {
        "status": "success"
    }
```

## DELETE /api/v1/user/friends/{username}

```
    {
        "status": "success"
    }
```

---

# Connections

## POST /api/v1/user/connections
## DELETE /api/v1/user/connections/{username}
## GET /api/v1/user/connection_request

## GET /api/v1/users/connections

```
    {
        "status": "success",
        "data": {
            "users": [
                {
                    "username": string,
                    "connection": string,   // either "friend" or "connection"
                    "images": {
                        "thumbnail": URLstring,
                        "hero": URLstring,
                        "tiny": URLstring
                    },
                    "location": {
                        "node_id": locationNodeId,
                        "root": {
                            "id": stringDomainId,
                            "name": stringDomainName,
                            "network_address": stringNetworkAddress,
                            "network_port": numberPort,
                            "ice_server_address": stringNetworkAddress,
                            "time_of_last_heartbeat": stringISODate,
                            "num_users": numberUsers
                        },
                        "path": stringAddress,
                        "online": boolean
                    }
                },
                ...
            ]
        }
    }
```

---

# Administrative

## PUT /api/v1/user/location
## GET /api/v1/users/{username}/location

The `username` is percent-encoded for inclusion to the URL.

```
    {
        "status": "success",
        "data": {
            "location": {   // can be "place"
                "root": {
                    "domain": {
                        "id":
                        "network_address":
                        "network_port":
                        "ice_server_address":
                        "name":
                    },
                    "name": placeName,
                },
                "path": "/X,Y,Z/X,Y,Z,W",
                "online": bool
            }
        }
    }
```

## PUT /api/v1/user/public_key
## GET /api/v1/users/{username}/public_key

The `username` is percent-encoded for inclusion to the URL.

## POST /api/v1/users

Create user account.

```
    {
        "username": "name",
        "password": "password",
        "email": "email@example.com"
    }
```

On successful account creation:

```
    {
        "status": "success",
        "data": {
            "accountId": "id",
            "username": "name",
            "accountIsActive": true,
            "accountWaitingVerification": false
        }
    }
```

Note that, if account email validation is enabled, the account will be created
but it will not be able to create access tokens until verified.
If awaiting verification, `accountWaitingVerification` will be `true`
and `accountIsActive` will be `false`.

If creation failed:

```
    {
        "status": "failure",
        "error": "Reason for failure"
    }
```

## PUT /api/v1/user/heartbeat


```
    {
        "location": {
        }
    }
```

```
    {
        "status": "success",
        "data": {
            "session_id": stringSessionId
        }
    }
```


## GET /api/v1/user/locker

## POST /api/v1/user/locker

