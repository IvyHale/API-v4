## OVERVIEW
The Virgil Cards Service is the core service of the Virgil infrastructure and it's responsible for the Virgil Card entries management.
The main purpose of the Virgil Card entry is to store a user's Public Key with an optional identity which is issued on the application developer's side.

Card Service address:

https://cards.virgilsecurity.com/v4


## AUTHORIZATION
The Authorization HTTP header is mandatory for API calls that are related to the Virgil Cards with scope = "application" and is skipped for Virgil Cards with scope = "global". An access tokens for every application can be retrieved on the Virgil Development Portal.
The Authorization Header:

```
GET /v4/card/a6f7b874ea69329372ad75353314d7bc...
Host: cards.virgilsecurity.com
Authorization: VIRGIL { YOUR_APPLICATION_TOKEN }
```

Authorization has the following parameters:
|Parameter|Description|
|---|---|
|Authorization: required|This parameter indicates that the endpoint needs an authorization.|
|signs|This parameter is nested into the meta request parameter, an associative array that contains the signer's fingerprint, which is represented as Keys and Base64-encoded signs and values.|
|Fingerprint|This identifies a specific Virgil Card.|

The signs parameter has the following structure:
```
{
    "meta": {
        "signs": {
          "[CARD_HOLDER_ID]": "[VIRGIL_CARD_BASE64_ENCODED_SIGN_OF_THE_FINGERPRINT]",
          "[APPLICATION_ID]": "[VIRGIL_CARD_BASE64_ENCODED_SIGN_OF_THE_FINGERPRINT]"
        }
     }
}
```

## CARD STRUCTURE
Virgil Card has the following parameters:

|Parameter|Description|
|---|---|
|public_key*|This Parameter must contain a Base64-encoded public key value in DER or PEM format.|
|scope*|This parameter determines a Virgil Card's scope that can be either "global" or "application". Application Virgil Cards are accessible only within the application they were created within. Global Virgil Cards are available in all the applications.|
|identity_type*|This parameter must be "email" for a confirmed Virgil Card and can be any value for a segregated one.|
|identity*|Must be a valid email for a confirmed Virgil Card with an identity_type of "email" and can be any value for a segregated one.|
|data|This parameter is an associative array that contains application specific parameters. All keys must contain static characters and digits only. The length of keys and values must not exceed 256 characters. Please note that more than 16 data items cannot be persisted at a time.|
|info|This parameter is an associative array with predefined keys that contain information about the device on which the keypair was created. The keys are always device_name and device and the values must not exceed 256 characters. Both keys are optional but at least one of them must be specified, if the info parameter is specified.|
|signs*|This parameter is mandatory to authorize the creation of a Virgil Card by the Virgil Card holder or by the authorized application. The signs parameter must always contain either the Virgil Card holder's signature, the application signature, or the Virgil Registration Authority Service signature (or both). The signed_digest is calculated as a BASE64_ENCODE(SIGN(FINGERPRINT, PRIVATE_KEY)). The private key must belong to the Virgil Card holder, the Application, or the Virgil Registration Authority service. The signs parameter must always contain signature of the Virgil Registration Authority service to create a Virgil Card with scope = "global".|
|content_snapshot|This is a Base64-encoded string with a JSON representation of the Virgil Card data required for an operation '(contentSnapshot = BASE64(virgilCardJsonData))'. The content_snapshot will be persisted alongside the Virgil Card, and can't be changed during the Virgil Card's lifetime. It can be used by the Virgil Card's owner and the application service, to make sure the Virgil Card's data was not changed by a 3rd-party.|
|created_at|This parameter shows the time that the Virgil Card was created.|
|card_version|Virgil Card Version|
|id|Virgil Card ID|
|relations|This relation entity describes a trusted, one-way relationship between the source Virgil Card specified in the URI, and the destination Virgil Card which the CSR is specified to, in the request body.|
* — These parameters are mandatory

Example:
```
public_key    = "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTULS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTU.LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTU.LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTU.";
identity_type = "email";
identity      = "user@VirgilSecurity.com";
scope         = "global";
data          = {
    "custom_key_1": "custom_value_1",
    "custom_key_2": "custom_value_2"
};
info = {
    "device": "iPhone6s",
    "device_name": "Space grey one"
};
```

Each Virgil Card is created by passing the contentSnapshot, which contains all data related to the Virgil Card, and is represented as a JSON.
The JSON representation:

```
requestJsonData = {       
    "public_key":"LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0t...",
    "identity_type":"email",
    "identity":"user@VirgilSecurity.com",
    "scope":"global",
    "data":{"custom_key_1":"custom_value_1",
            "custom_key_2":"custom_value_2"},
    "info":{"device":"iPhone6s",
            "device_name":"Space grey one"}}
```

The content snapshot will persist alongside the Virgil Card, and is not supposed to be changed during the Virgil Card's lifetime. It can be used by the Virgil Card's owner and the application service to make sure the Virgil Card's data was not changed by a 3rd-party.

```
contentSnapshot       = BASE64(virgilCardJsonData)
virgilCardFingerprint = SHA256(virgilCardJsonData)
virgilCardId          = HEX(virgilCardFingerprint)
```


## CREATE CARD ENDPOINT

The endpoint creates a Virgil Card entity.
The Virgil Card creation action requires an authorization header from the Virgil Card owner which is performed via the additional signatures parameter. It must be an associative array with all necessary information about card signers.

### Request Info
```
HTTP Request method    POST
Request URL            https://cards.virgilsecurity.com/v4/card
Authorization          Required
```

### Request Body
```
{
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiTFMwdExTMU...",
    "meta": {
        "signs": {
            "bb5db5084dab...":"MIGaMA0GCWCGSAFl...",
            "767b6b12702d...":"MIGaMA0GCWCGSAFl..."
        }
    }
}
```
### Response Body
```
{
    "id": "bb5db5084dab51113...",
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiT...",
    "meta": {
        "created_at": "2015-12-22T07:03:42+0000",
        "card_version": "4.0",
        "signs": {
            "bb5db5084dab51...":"MIGaMA0GCWCGSAFl...",
            "767b6b12702df1...":"MIGaMA0GCWCGSAFl...",
            "ab799a2f26333c...":"MIGaMA0GCWCGSAFl..."
        },
        "relations": {
            "f2ac09330139a2...":"MIGaMA0GCWCGSAFl..."
        }
    }
}
```
To create a confirmed Virgil Card, it's necessary to delegate the card's creation to the Virgil RA service. The Virgil Card will be marked as confirmed as long as the Virgil Identity sign was passed.
In order to create an unconfirmed, segregated Virgil Card, one must simply set the scope request parameter to "application" and pass a valid application sign item in the signs list;
Beware that to create a Global Virgil Card, it's mandatory to perform a call to the Virgil Registration Authority service instead of the Virgil Cards service.
The request that creates a Virgil Card contains two signed items: for the application holder and for the application.
After the Virgil Card's endpoint invocation, the signs list is filled with an additional Virgil Cards service sign. All Virgil Card data is passed in the content_snapshot parameter, then the Virgil Cards service creates an additional sign item, with its own fingerprint used as a key to prove that it really created the Virgil Card.

## GET CARD ENDPOINT
Returns the information about the Virgil Card by the its ID.

### Request Info

```
HTTP Request method    GET
Request URL  
https://cards-ro.virgilsecurity.com/v4/card/{card-id}
```

### Response Body

```
{
    "id": "bb5db5084dab511135ec24c....",
    "content_snapshot":"eyJwdWJsaWNfa2V5...",
    "meta": {
        "created_at": "2015-12-22T07:03:42+0000",
        "card_version": "4.0",
        "signs": {
            "af6799a2f26376731a...":"MIGaMA0GCWCGSAFl...",
            "767b6b12702df1a873...":"MIGaMA0GCWCGSAFl...",
            "ab799a2f26333c09af...":"MIGaMA0GCWCGSAFl..."

        },
        "relations": {
            "f2ac09330139a2f263...":"MIGaMA0GCWCGSAFl..."
        }
    }
}
```

## SEARCH CARDS ENDPOINT
This endpoint shows how to perform a Virgil Card search by some criteria.

### Request Info
```
HTTP Request method    POST
Request URL            
https://cards-ro.virgilsecurity.com/v4/card/actions/search
```

### Request Body

```
{
    "identities": ["user@VirgilSecurity.com",
                   "another.user@VirgilSecurity.com"],
    "identity_type": "email",
    "scope": "global"
}
```

### Response Body

```
[
    {
        "id": "bb5db5084dab511135ec24c2fdc5...",
        "content_snapshot":"eyJwdWJsaWNfa2V5Ij...",
        "meta": {
            "created_at": "2015-12-22T07:03:42+0000",
            "card_version": "4.0",
            "signs": {
                "af6799a2f26376731...":"MIGaMA0GCWCGSAFlA...",
                "767b6b12702df1a87...":"MIGaMA0GCWCGSAFlA...",
                "ab799a2f26333c09a...":"MIGaMA0GCWCGSAFlA..."
            },
            "relations": {
                "f2ac09330139a2f26...":"MIGaMA0GCWCGSAFlA..."
            }
        }
    }
]
```

### Parameters:
|Parameter|Description|
|---|---|
|identities*|This parameter is an email of a confirmed Virgil Card.|
|identity_type|This is an optional request parameter, which specifies the identity_type of a Virgil Card to be found.|
|scope|This optional request parameter specifies the scope to perform the search on either "global" or "application". The default value is "application".|
* — These parameters are mandatory


## DELETE CARD
This endpoint is used to revoke a Virgil Card.

### Request Info

```
HTTP Request method    DELETE
Request URL            https://cards.virgilsecurity.com/v4/card/{card-id}
Authorization          Required
```

### Request Body
```
{
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiTFMwdE...",
    "meta": {
        "signs": {
            "af6799a2f26376731a...":"MIGaMA0GCWCGSAFlA..."
        }
    }
}
```

The content_snapshot parameter is a JSON representation that contains the following parameters:
```
{
    "card_id": "af6799a2f26376731abb9abf32b5f2...",
    "revocation_reason": "unspecified"
}
```

### Response Body
```
some code?
```

### Parameters:
|Parameter|Description|
|---|---|
|id|This parameter is the id of the Virgil Card to be removed.|
|revocation_reason|Contains one of the values from the list: "unspecified" or "compromised".|
|meta.signs|This parameter must contain an entry with the application sign for all the Virgil Cards which have their scope value set to "application"|

To revoke a Virgil Card with scope = "global" , it's mandatory to pass both the Client and the Virgil Registration Authority sign.

## CREATE CARD RELATION
This endpoint shows how to add a relation to a Virgil Card. The relation entity describes a trusted one-way relationship between the source Virgil Card specified in the URI and the destination Virgil Card, of which the CSR is specified in the request body.
Relations work similarly to the signs mechanism, but specifies the opposite trust direction. If the signs collection lists the Virgil Cards that were verified by the source Virgil Card at the moment of its creation, the relations collection will list all the Virgil Cards trusted by the source Virgil Card.

### Request info
```
HTTP Request method    POST
Request URL            https://cards.virgilsecurity.com/v4/card/{card-id}/collections/relations
Authorization          Required
```

### Request Body
```
{
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiTFM...",
    "meta": {
        "signs": {
            "af6799a2f2637673...":"MIGaMA0GCWCGSAFlA..."
        }
    }
}
```

### Response Body

```
{
    "id": "bb5db5084dab511135ec24c...",
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiTFMwdEx...",
    "meta": {
        "created_at": "2015-12-22T07:03:42+0000",
        "card_version": "4.0",
        "signs": {
            "bb5db5084dab511135ec...":"MIGaMA0GCWCGSAFlA...",
            "767b6b12702df1a873f4...":"MIGaMA0GCWCGSAFlA...",
            "ab799a2f26333c09af66...":"MIGaMA0GCWCGSAFlA..."
        },
        "relations": {
            "f2ac09330139a2f26376...":"MIGaMA0GCWCGSAFlA..."
        }
    }
}
```

### Parameters:
|Parameter|Description|
|---|---|
|content_snapshot|The CSR of the destination Virgil Card. This means that it's required to pass a value already passed.|
|meta.signs|The collection with one mandatory item where the key must be the same Virgil Card ID that is specified in the URI. Also, the value must be the Base64-encoded digital sign of the content_snapshot by the source Virgil Card private key.|

## DELETE CARD RELATION
This endpoint shows how to remove a relation from the Virgil Card.

### Request Info
```
HTTP Request method    DELETE
Request URL            https://cards.virgilsecurity.com/v4/card/{card-id}/collections/relations
Authorization          Required
```

### Request Body
```
{
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiTFMwdExTMUNS...",
    "meta": {
        "signs": {
            "af6799a2f26376731a...":"MIGaMA0GCWCGSAFlA..."
        }
    }
}
```
The content_snapshot parameter is a JSON representation that contains the following parameters:
```
{
    "card_id": "af6799a2f26376731abb9abf...",
    "revocation_reason": "unspecified"
}
```

### Response Body
```
{
    "id": "bb5db5084dab511135...",
    "content_snapshot":"eyJwdWJsaWNfa2V5IjoiTF...",
    "meta": {
        "created_at": "2015-12-22T07:03:42+0000",
        "card_version": "4.0",
        "signs": {
            "bb5db5084dab511135ec2...":"MIGaMA0GCWCGSAFlAw...",
            "767b6b12702df1a873f42...":"MIGaMA0GCWCGSAFlAw...",
            "ab799a2f26333c09af662...":"MIGaMA0GCWCGSAFlAw..."
        },
        "relations": {
        }
    }
}
```

### Parameters:
|Parameter|Description|
|---|---|
|id	|This parameter is the id of the Virgil Card with the relation that is supposed to be removed.|
|revocation_reason|This is the optional parameter with the code from the list: "unspecified" or "compromised"|
|meta.signs	|The collection with one mandatory item where the key must be the same Virgil Card id that is specified in the URI. Also, the value must be the Base64-encoded digital sign of the content_snapshot by the source Virgil Card private key.|


## ERRORS
The Application uses standard HTTP response codes:
|Error|Description|
|---|---|
|200	|Success|
|400	|Request error|
|401	|Authentication error|
|403	|Forbidden|
|404	|Entity not found|
|405	|Method not allowed|
|500	|Server error|

### HTTP 500. Server error
This status is returned in extremely rare cases of internal application errors
|Error|Description|
|---|---|
|10000	| Internal application error|

### HTTP 401. Auth error
This status is returned on authorization errors
|Error|Description|
|---|---|
|20300	|The Virgil access token or token header was not specified or is invalid|
|20301	|The Virgil authenticator service responded with an error|
|20302	|The Virgil access token validation has failed on the Virgil Authenticator service|
|20303	|The Application was not found for the access token|

### HTTP 403. Forbidden
This status is returned when a request is not granted permission to the resource
|Error|Description|
|---|---|
|20500	|The Virgil Card is not available in this application|

### HTTP 400. Request error
|Error|Description|
|---|---|
|30000	|JSON specified as a request is invalid|
|30010	|A data inconsistency error|
|30100	|Global Virgil Card identity type is invalid, because it can be only an 'email'|
|30101	|Virgil Card scope must be either 'global' or 'application'|
|30102	|Virgil Card ID validation failed|
|30103	|Virgil Card data parameter cannot contain more than 16 entries|
|30104	|Virgil Card info parameter cannot be empty if specified and must contain 'device' and/or the 'device_name' key|
|30105	|Virgil Card info parameters length validation failed. The value must be a string and must not exceed 256 characters|
|30106	|Virgil Card data parameter must be a key/value list of strings|
|30107	|A CSR parameter (content_snapshot) parameter is missing or is incorrect|
|30111	|Virgil Card identities passed to search endpoint must be a list of non-empty strings|
|30113	|Virgil Card identity type is invalid|
|30114	|Segregated Virgil Card custom identity value must not be an empty string|
|30115	|Virgil Card identity email is invalid|
|30116	|Virgil Card identity application is invalid|
|30117	|Public key length is invalid. Public keys go from 16 to 2048 bytes|
|30118	|Public key must be Base64-encoded string|
|30119	|Virgil Card data parameter must be a key/value list of strings|
|30120	|Virgil Card data parameters must be strings|
|30121	|Virgil Card custom data entry value length validation failed. It must not exceed 256 characters|
|30123	|SCR signs list parameter is missing or is invalid|
|30128	|SCR sign item is invalid or missing for the application|
|30131	|Virgil Card id specified in the request body must match with the one passed in the URL|
|30134	|Virgil Card data parameters key must be alphanumeric|
|30138	|Virgil Card with the same fingerprint exists already|
|30139	|Virgil Card revocation reason is not specified or is invalid|
|30140	|SCR sign validation failed|
|30141	|SCR one of signers Virgil Cards is not found|
|30142	|SCR sign item is invalid or missing for the Client|
|30143	|SCR sign item is invalid or missing for the Virgil Registration Authority service|
|30200	|Virgil Card relation sign is invalid|
|30201	|Virgil Card relation sign by the source Virgil Card was not found|
|30202	|Related Virgil content snapshot parameter was not found|
|30203	|The relation with this Virgil Card exists already|
|30204	|The related Virgil Card was not found for the provided CSR|
|30205	|The Virgil Card relation does not exist|

Additional information about the error is returned as JSON-object like:
```
{
    "code": "{error-code}"
}
```
