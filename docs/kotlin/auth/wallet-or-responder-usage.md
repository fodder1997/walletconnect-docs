# Wallet / Responder Usage

We recommend looking at example implementations of Responder at our [Kotlin Github repository](https://github.com/WalletConnect/WalletConnectKotlinV2/tree/develop/auth/responder)

### **Initialize Auth Client**

To initialize the Auth client, create a `Auth.Params.Init` object in the Android Application class. The Init object will need the
application class, the Project ID, and the application AppMetaData. The `Auth.Params.Init` object will then be passed to the `AuthClient`
`initialize` function. `Auth.Params.Init` also allows for custom URLs by passing URL string into the `relayUrl` property. `iss` parameter must not be null for Wallet / Responder, it describes what did responder authorises. Example `iss` for Ethereum Mainnet: `did:pkh:eip155:1:0xb9c5714089478a327f09197987f16f9e5d936e8a`. More about `did:pkh` method [here](https://github.com/w3c-ccg/did-pkh/blob/main/did-pkh-method-draft.md)

```kotlin
val projectId = "" // Get Project ID at https://cloud.walletconnect.com/
val relayUrl = "relay.walletconnect.com"
val serverUrl = "wss://$relayUrl?projectId=${projectId}"
val appMetaData = Core.Model.AppMetaData(name = "Kotlin.Responder",
    description = "Kotlin AuthSDK Responder Implementation",
    url = "kotlin.responder.walletconnect.com",
    icons = listOf("https://raw.githubusercontent.com/WalletConnect/walletconnect-assets/master/Icon/Gradient/Icon.png"),
    redirect = "kotlin-responder-wc:/request"
)

CoreClient.initialize(relayServerUrl = serverUrl, connectionType = ConnectionType.AUTOMATIC, application = this,metaData = appMetaData)

AuthClient.initialize(
    init = Auth.Params.Init(
        core = CoreClient,
        iss = ISSUER
    )
) { error ->
    Log.e(tag(this), error.throwable.stackTraceToString())
}
```
For more contex on how to initialize CoreClient, go to [CoreClient docs](../../kotlin/core/installation.md) section.

---
### **AuthClient.ResponderDelegate**

The AuthClient needs a `AuthClient.ResponderDelegate` passed to it for it to be able to expose asynchronously updates sent from the Dapp / Requester.

```kotlin
object ResponderDelegate : AuthClient.ResponderDelegate {
    init {
        AuthClient.setResponderDelegate(this)
    }

    override fun onAuthRequest(authRequest: Auth.Event.AuthRequest) {
        // Triggered when Dapp / Requester makes an authorisation request. Wallet / Responder should display message to user and ask him to approve or reject authorisation.
    }

    override fun onConnectionStateChange(connectionStateChange: Auth.Event.ConnectionStateChange) {
        // Triggered whenever the connection state is changed
    }

    override fun onError(error: Auth.Event.Error) {
        //Triggered whenever the error occurs with Relay Server
    }
}
```

---

## **Methods**

### **Authotisation Request Approval**

To approve authorisation request, sign message using `CacaoSigner.sign` which requires private key to sign `Cacao` object that needs to be passed to `Auth.Params.Respond.Result` object and send to Dapp / Requester.

```kotlin
val request: Auth.Event.AuthRequest = // Request from onAuthRequest
val cacao: Auth.Model.Cacao.Signature = CacaoSigner.sign(
    request.message, // Message to be signed
    PRIVATE_KEY, // Private key used to signing a message
    SignatureType.EIP191 // Currently only EIP191 signing is supported
)

AuthClient.respond(Auth.Params.Respond.Result(request.id, cacao)) { error ->
    Log.e("Responder respond approve", error.throwable.stackTraceToString())
}
```

### **Authorisation Request Rejection**

To reject authorisation request respond Dapp / Requester with `Auth.Params.Respond.Error`. Note: We recommend using defined below error message and error code.

```kotlin
val request: Auth.Event.AuthRequest = // Request from onAuthRequest
AuthClient.respond(
    Auth.Params.Respond.Error(request.id, 12001, "User Rejected Request") // Specifying Error codes will change in future
) { error ->
    Log.e("Responder respond reject", error.throwable.stackTraceToString())
}
```


### **Get List of Pending Requests**

```kotlin
val pendingRequest: List<Auth.Model.PendingRequest> = AuthClient.getPendingRequest()

```

To get a list of pending request, call `AuthClient.getPendingRequest()` which will return a list of type `Auth.Model.PendingRequest`.

