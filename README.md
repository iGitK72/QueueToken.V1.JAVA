[ ![Download](https://api.bintray.com/packages/queueit/maven/com.queue_it.queuetoken/images/download.svg) ](https://bintray.com/queueit/maven/com.queue_it.queuetoken/_latestVersion)

# Queue-it Queue Token SDK for JAVA 
The Queue-it Queue Token SDK is used to ensure that end users cannot enter the queue without a valid token and to be a container which can car-ry sensitive user information from integrating system into the queue. The token can be issued by any application that supports JAVA 1.6+.
## The Token
The token consists of two parts. Firstly, a header containing non-sensitive metadata. Secondly the payload of the token. Both header and payload are in JSON format.
### Token Header
```
{ 
  "typ": "QT1",
  "enc": "AES256",
  "iss": 1526464517,
  "exp": 1526524517,
  "ti": "159aba3e-55e1-4f54-b6ee-e5b943d7e885",
  "c": "ticketania", 
  "e": "demoevent",
  "ip": "75.86.129.4",
  "xff": "45.67.2.4,34.56.3.2"
}
```
- `typ`: The type of the token. Value must be "QFT1". Required.
- `enc`: Payload encryption algorithm. Value must be "AES256". Required.
- `iss`: NumericDate of when token was issued. Required.
- `exp`: NumericDate of when token expires. Optional.
- `ti`: Unique Token ID (e.g. uuid). Used to uniquely identify tokens and restrict replay attacks. Required.
- `c`: The Customer ID of the issuer. Token will only be valid on events on this account. Required.
- `e`: The Event ID. If provided, token will only be valid on this event. Optional.
- `ip`: The IP address of user the token is issued for. If provided, the IP address in the token is validated against the client IP before issuing a new Queue id. Optional.
- `xff`: The X-Forwarded-For header of the request when the token is issued. Only used for logging. Optional.

### Token Payload
```
{ 
  "r": 0.4578,
  "k": "XKDI42W",
  "cd": { "size": "medium" }
}
```
- `r`: The relative quality of the key. Must be a decimal value. Used for determining the quality of the token. Optional
- `k`: A unique key that holds value to the integrating system (e.g. email or user id). Used to restrict users from issuing multiple queue ids. Optional.
- `cd`: Any custom data of the user. This is a set of key-value pairs. Optional

## Usage
```
String secretKey = ...;
IEnqueueToken token = Token
  .enqueue("ticketania")
  .withPayload(Payload
    .enqueue()
    .withKey("XKDI42W")
    .WithRelativeQuality(0.4578)
    .withCustomData("size", "medium")
    .qenerate())
  .withEventId("demoevent")
  .withIpAddress("75.86.129.4", "45.67.2.4,34.56.3.2")
  .withValidity(60000)
  .generate(secretKey);

String token = token.getToken();
```

### Specifying token identifier prefix
A prefix for the token identifier can optionally be provided to restrict the user session after getting through the queue to the one used before entering the queue. Once the user is through the queue the token identifier is provided to the target application in the Known User token. The format of the token identifier is then `[YOUR PREFIX]~[GUID]`, e.g: AnfTDnpwazllYmnmgaCJ8tErV80YHv77ni5NgqQNhfWwxNqrNcHb~e937ef0d-48ec-4ff7-866e-52033273cb3d.
```
String tokenIdentifierPrefix = "AnfTDnpwazllYmnmgaCJ8tErV80YHv77ni5NgqQNhfWwxNqrNcHb";
IEnqueueToken token = Token
  .enqueue("ticketania", tokenIdentifierPrefix)
  .generate(secretKey);

String tokenIdentifier = token.getTokenIdentifier();
// tokenIdentifier example: AnfTDnpwazllYmnmgaCJ8tErV80YHv77ni5NgqQNhfWwxNqrNcHb~e937ef0d-48ec-4ff7-866e-52033273cb3d
```

## Serialized Token
> eyJ0eXAiOiJRVDEiLCJlbmMiOiJBRVMyNTYiLCJpc3MiOjE1MzQ3MjMyMDAwMDAsImV4cCI6MTUzOTEyOTYwMDAwMCwidGkiOiJhMjFkNDIzYS00M2ZkLTQ4MjEtODRmYS00MzkwZjZhMmZkM2UiLCJjIjoidGlja2V0YW5pYSIsImUiOiJteWV2ZW50In0.0rDlI69F1Dx4Twps5qD4cQrbXbCRiezBd6fH1PVm6CnVY456FALkAhN3rgVrh_PGCJHcEXN5zoqFg65MH8WZc_CQdD63hJre3Sedu0-9zIs.aZgzkJm57etFaXjjME_-9LjOgPNTTqkp1aJ057HuEiU

The format of the token is [header].[payload].[hash] where each part is Base64Url encoded. The payload is AES 256 encrypted with the secret key supplied in the `.Generate(secretKey)` method. If the "e" key is provided in the header, the secret key on the event must be used. If no "e" key is provided the default key on the customer account must be used.
The token is signed with SHA 256 using the same secret key.

## Trouble Shooting
### Java Cryptography Extension (JCE) unlimited strength jurisdiction policy files
Due to import control restrictions by the governments of a few countries, the jurisdiction policy files shipped with the JDK from Sun Microsystems specify that "strong" but limited cryptography may be used. This causes java.security.InvalidKeyException (Illegal key size or default parameters). Please refer to http://opensourceforgeeks.blogspot.com/2014/09/how-to-install-java-cryptography.html
