# Authentication

The server has builtin authentication so clients can only send requests and connect if they have the [server key](../getting-started/configuration.md#socket). When authentication is successful a client can create a session as a [user](user-accounts.md).

!!! Warning "Important"
    The default server key is `defaultkey` but it is very important to set a [unique value](../getting-started/configuration.md#socket). This value should be embedded within client code.

=== "JavaScript"
    ```js
    var client = new nakamajs.Client("defaultkey", "127.0.0.1", 7350);
    client.ssl = false;
    ```

=== ".NET"
    ```csharp
    // Use "https" scheme if you've setup SSL.
    var client = new Client("http", "127.0.0.1", 7350, "defaultkey");
    ```

=== "Unity"
    ```csharp
    // Use "https" scheme if you've setup SSL.
    var client = new Client("http", "127.0.0.1", 7350, "defaultkey");
    ```

=== "Cocos2d-x C++"
    ```cpp
    NClientParameters parameters;
    parameters.serverKey = "defaultkey";
    parameters.host = "127.0.0.1";
    parameters.port = DEFAULT_PORT;
    NClientPtr client = NCocosHelper::createDefaultClient(parameters);
    ```

=== "Cocos2d-x JS"
    ```js
    var serverkey = "defaultkey";
    var host = "127.0.0.1";
    var port = 7350;
    var useSSL = false;
    var timeout = 7000; // ms

    var client = new nakamajs.Client(serverkey, host, port, useSSL, timeout);
    ```

=== "C++"
    ```cpp
    NClientParameters parameters;
    parameters.serverKey = "defaultkey";
    parameters.host = "127.0.0.1";
    parameters.port = DEFAULT_PORT;
    NClientPtr client = createDefaultClient(parameters);
    ```

=== "Java"
    ```java
    Client client = new DefaultClient("defaultkey", "127.0.0.1", 7349, false)
    // or same as above.
    Client client = DefaultClient.defaults("defaultkey");
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let client : Client = Builder("defaultkey")
        .host("127.0.0.1")
        .port(7350)
        .ssl(false)
        .build()
    // or same as above.
    let client : Client = Builder.defaults(serverKey: "defaultkey")
    ```

=== "Godot"
    ```gdscript
    onready var client := Nakama.create_client("defaultkey", "127.0.0.1", 7350, "http")
    ```

=== "Defold"
    ```lua
    local defold = require "nakama.engine.defold"

    local config = {}
    config.host = 127.0.0.1
    config.port = 7350
    config.use_ssl = false
    config.username = "defaultkey"
    config.engine = defold

    local client = nakama.create_client(config)
    ```

Every user account is created from one of the [options used to authenticate](#authenticate). We call each of these options a "link" because it's a way to access the user's account. You can add more than one link to each account which is useful to enable users to login in multiple ways across different devices.

## Authenticate

Before you interact with the server, you must obtain a session token by authenticating with the system. The authentication system is very flexible. You could register a user with an email address, [link](#link-or-unlink) their Facebook account, and use it to login from another device.

!!! Note
    By default the system will create a user automatically if the identifier used to authenticate did not previously exist in the system. This pattern is shown in the [device](#device) section.

For a __full example__ on the best way to handle register and login in each of the clients have a look at their guides.

### Device

A device identifier can be used as a way to unobtrusively register a user with the server. This offers a frictionless user experience but can be unreliable because device identifiers can sometimes change in device updates.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

A device identifier must contain alphanumeric characters with dashes and be between 10 and 128 bytes.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/device?create=true&username=mycustomusername" \
      --user 'defaultkey:' \
      --data '{"id":"uniqueidentifier"}'
    ```

=== "JavaScript"
    ```js
    // This import is only required with React Native
    var deviceInfo = require('react-native-device-info');

    var deviceId = null;
    try {
      const value = await AsyncStorage.getItem('@MyApp:deviceKey');
      if (value !== null){
        deviceId = value
      } else {
        deviceId = deviceInfo.getUniqueID();
        AsyncStorage.setItem('@MyApp:deviceKey', deviceId).catch(function(error){
          console.log("An error occured: %o", error);
        });
      }
    } catch (error) {
      console.log("An error occured: %o", error);
    }

    var create = true;
    const session = await client.authenticateDevice(deviceId, create, "mycustomusername");
    console.info("Successfully authenticated:", session);
    ```

=== ".NET"
    ```csharp
    // Should use a platform API to obtain a device identifier.
    var deviceId = System.Guid.NewGuid().ToString();
    var session = await client.AuthenticateDeviceAsync(deviceId);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    var deviceId = PlayerPrefs.GetString("nakama.deviceid");
    if (string.IsNullOrEmpty(deviceId)) {
        deviceId = SystemInfo.deviceUniqueIdentifier;
        PlayerPrefs.SetString("nakama.deviceid", deviceId); // cache device id.
    }
    var session = await client.AuthenticateDeviceAsync(deviceId);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto loginFailedCallback = [](const NError& error)
    {
    };

    auto loginSucceededCallback = [](NSessionPtr session)
    {
        CCLOG("Successfully authenticated");
    };

    std::string deviceId = "unique device id";
    client->authenticateDevice(
            deviceId,
            opt::nullopt,
            opt::nullopt,
            {},
            loginSucceededCallback,
            loginFailedCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    var deviceId = "unique device id";
    var create = true;
    client.authenticateDevice(deviceId, create, "mycustomusername")
      .then(function(session) {
            cc.log("Authenticated successfully. User id:", session.user_id);
        },
        function(error) {
            cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto loginFailedCallback = [](const NError& error)
    {
    };

    auto loginSucceededCallback = [](NSessionPtr session)
    {
        cout << "Successfully authenticated" << endl;
    };

    std::string deviceId = "unique device id";

    client->authenticateDevice(
            deviceId,
            opt::nullopt,
            opt::nullopt,
            {},
            loginSucceededCallback,
            loginFailedCallback);
    ```

=== "Java"
    ```java
    String id = UUID.randomUUID().toString();
    Session session = client.authenticateDevice(id).get();
    System.out.format("Session: %s ", session.getAuthToken());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let defaults = UserDefaults.standard
    let deviceKey = "device_id"

    var deviceId : String? = defaults.string(forKey: deviceKey)
    if deviceId == nil {
        deviceId = UIDevice.current.identifierForVendor!.uuidString
        defaults.set(deviceId!, forKey: deviceKey)
    }

    let message = AuthenticateMessage(device: deviceId!)
    client.login(with: message).then { session in
        print("Login successful")
    }.catch{ err in
        if (err is NakamaError) {
            switch err as! NakamaError {
            case .userNotFound(_):
                let _ = self.client.register(with: message)
                return
            default:
                break
            }
        }
        print("Could not login: %@", err)
    }
    ```

=== "Godot"
    ```gdscript
    # Unique ID is not supported by Godot in HTML5, use a different way to generate an id, or a different authentication option.
    var deviceid = OS.get_unique_id()
    var session : NakamaSession = yield(client.authenticate_device_async(deviceid), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/device?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
        "id": "uniqueidentifier"
    }
    ```

=== "Defold"
    ```lua
    local body = nakama.create_api_account_device(defold.uuid())
    -- login using the token and create an account if the user
    -- doesn't already exist
    local result = nakama.authenticate_device(client, body, true)
    if not result.token then
        print("Unable to login")
        return
    end
    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

In games it is often a better option to use [Google](#google) or [Game Center](#game-center) to unobtrusively register the user.

### Email

Users can be registered with an email and password. The password is hashed before it's stored in the database server and cannot be read or "recovered" by administrators. This protects a user's privacy.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

An email address must be valid as defined by RFC-5322 and passwords must be at least 8 characters.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/email?create=true&username=mycustomusername" \
      --user 'defaultkey:' \
      --data '{"email":"email@example.com", "password": "3bc8f72e95a9"}'
    ```

=== "JavaScript"
    ```js
    const email = "email@example.com";
    const password = "3bc8f72e95a9";
    const create = true;
    const session = await client.authenticateEmail(email, password, create, "mycustomusername");
    console.info("Successfully authenticated:", session);
    ```

=== ".NET"
    ```csharp
    const string email = "email@example.com";
    const string password = "3bc8f72e95a9";
    var session = await client.AuthenticateEmailAsync(email, password);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    const string email = "email@example.com";
    const string password = "3bc8f72e95a9";
    var session = await client.AuthenticateEmailAsync(email, password);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
      CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string email = "email@example.com";
    string password = "3bc8f72e95a9";
    string username = "mycustomusername";
    bool create = true;
    client->authenticateEmail(email, password, username, create, {}, successCallback, errorCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const email = "email@example.com";
    const password = "3bc8f72e95a9";
    client.authenticateEmail(email, password, true, "mycustomusername")
      .then(function(session) {
          cc.log("Authenticated successfully. User ID:", session.user_id);
        },
        function(error) {
          cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string email = "email@example.com";
    string password = "3bc8f72e95a9";
    string username = "mycustomusername";
    bool create = true;
    client->authenticateEmail(email, password, username, create, {}, successCallback, errorCallback);
    ```

=== "Java"
    ```java
    String email = "email@example.com";
    String password = "3bc8f72e95a9";
    Session session = client.authenticateEmail(email, password).get();
    System.out.format("Session: %s ", session.getAuthToken());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let email = "email@example.com"
    let password = "3bc8f72e95a9"

    let message = AuthenticateMessage(email: email, password: password)
    client.register(with: message).then { session in
        NSLog("Session: %@", session.token)
    }.catch { err in
        NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    // Use client.login(...) after register.
    ```

=== "Godot"
    ```gdscript
    var email = "email@example.com"
    var password = "3bc8f72e95a9"
    var session : NakamaSession = yield(client.authenticate_email_async(email, password, "mycustomusername", true), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/email?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
        "email": "email@example.com",
        "password": "3bc8f72e95a9"
    }
    ```

=== "Defold"
    ```lua
    local body = nakama.create_api_account_email("email@example.com", "3bc8f72e95a9")
    local result = nakama.authenticate_email(client, body, true, "mycustomusername")
    if not result.token then
        print("Unable to login")
        return
    end
    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

### Social providers

The server supports a lot of different social services with register and login. With each provider the user account will be fetched from the social service and used to setup the user. In some cases a user's [friends](friends.md) will also be fetched and added to their friends list.

To register or login as a user with any of the providers an OAuth or access token must be obtained from that social service.

#### Apple

Follow the Apple Developer documentation for integrating [Sign in with Apple](https://developer.apple.com/sign-in-with-apple/get-started/) in your applications.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/apple?create=true&username=mycustomusername" \
      --user 'defaultkey:' \
      --data '{"bundle_id":"...", "token":"..."}'
    ```

=== ".NET"
    ```csharp
    var bundleId = "...";
    var token = "...";

    var session = await client.AuthenticateAppleAsync(bundleId, token,
        publicKeyUrl, salt, signature, timestamp);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    // The official Unity Sign in with Apple package
    // has been deprecated from the Asset Store.
    // Alternative packages are available, such as
    // https://github.com/lupidan/apple-signin-unity#implement-sign-in-with-apple

    var bundleId = "...";
    var token = "...";

    var session = await client.AuthenticateAppleAsync(bundleId, token);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    auto errorCallback = [](const NError& error)
    {
    };

    std::string bundleId = "...";
    std::string token = "...";

    client->authenticateApple(bundleId, token);
    ```

=== "Cocos2d-x JS"
    ```js
    const bundle_id = "...";
    const token = "...";

    client.authenticateApple({
      bundle_id: bundle_id,
      token: token,
      username: "mycustomusername",
      create: true
    }).then(function(session) {
      cc.log("Authenticated successfully. User ID:", session.user_id);
    },

    function(error) {
      cc.error("authenticate failed:", JSON.stringify(error));
    });
    ```

=== "C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
    };

    auto errorCallback = [](const NError& error)
    {
    };

    std::string bundleId = "...";
    std::string token = "...";

    client->authenticateApple(bundleId, token);
    ```

=== "Godot"
    ```gdscript
    var bundle_id = "..."
    var token = "..."

    var session : NakamaSession = yield(client.authenticate_apple_async(bundle_id, token), "completed")

    if session.is_exception():
        print("An error occured: %s" % session)
        return

    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/apple?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
      "bundle_id": "...",
      "token": "...",
    }
    ```

=== "Defold"
    ```lua
    -- Use https://defold.com/assets/siwa/
    local bundle_id = "..."
    local token = "..."

    local body = nakama.create_api_account_apple(bundle_id, token)
    local result = nakama.authenticate_apple(client, body, true, "mycustomusername")

    if not result.token then
      print("Unable to login")
      return
    end

    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

#### Facebook

With Facebook you'll need to add the Facebook SDK to your project which can be <a href="https://developers.facebook.com/docs/" target="\_blank">downloaded online</a>. Follow their guides on how to integrate the code. With a mobile project you'll also need to complete instructions on how to configure iOS and Android.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

You can optionally import Facebook friends into Nakama's [friend graph](friends.md) when authenticating. To do this, set `import` to true.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/facebook?create=true&username=mycustomusername&import=true" \
        --user 'defaultkey:' \
        --data '{"token":"valid-oauth-token"}'
    ```

=== "JavaScript"
    ```js
    const oauthToken = "...";
    const import = true;
    const session = await client.authenticateFacebook(oauthToken, true, "mycustomusername", import);
    console.log("Successfully authenticated:", session);
    ```

=== ".NET"
    ```csharp
    const string oauthToken = "...";
    var session = await client.AuthenticateFacebookAsync(oauthToken);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    // using Facebook.Unity;
    // https://developers.facebook.com/docs/unity/examples#init
    var perms = new List<string>(){"public_profile", "email"};
    FB.LogInWithReadPermissions(perms, async (ILoginResult result) => {
        if (FB.IsLoggedIn) {
            var accessToken = Facebook.Unity.AccessToken.CurrentAccessToken;
            var session = await client.LinkFacebookAsync(session, accessToken);
            Debug.LogFormat("New user: {0}, {1}", session.Created, session);
        }
    });
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto loginFailedCallback = [](const NError& error)
    {
    };

    auto loginSucceededCallback = [](NSessionPtr session)
    {
        CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    std::string oauthToken = "...";
    bool importFriends = true;

    client->authenticateFacebook(
            oauthToken,
            "mycustomusername",
            true,
            importFriends,
            {},
            loginSucceededCallback,
            loginFailedCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const oauthToken = "...";
    const create = true;
    const import = true;
    client.authenticateFacebook(oauthToken, create, "mycustomusername", import)
      .then(function(session) {
            cc.log("Authenticated successfully. User ID:", session.user_id);
        },
        function(error) {
            cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto loginFailedCallback = [](const NError& error)
    {
    };

    auto loginSucceededCallback = [](NSessionPtr session)
    {
        cout << "Authenticated successfully. User ID: " << session->getUserId() << endl;
    };

    std::string oauthToken = "...";
    bool importFriends = true;
    client->authenticateFacebook(
            oauthToken,
            "mycustomusername",
            true,
            importFriends,
            {},
            loginSucceededCallback,
            loginFailedCallback);
    ```

=== "Java"
    ```java
    String oauthToken = "...";
    Session session = client.authenticateFacebook(oauthToken).get();
    System.out.format("Session %s", session.getAuthToken());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let oauthToken = "..."
    let message = AuthenticateMessage(facebook: oauthToken)
    client.register(with: message).then { session in
        NSLog("Session: %@", session.token)
    }.catch { err in
        NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    ```

=== "Godot"
    ```gdscript
    var oauth_token = "..."
    var session : NakamaSession = yield(client.authenticate_facebook_async(oauth_token), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/facebook?create=true&username=mycustomusername&import=true
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
        "token": "...",
    }
    ```

=== "Defold"
    ```lua
    -- Use the official Defold Facebook integration (www.defold.com/extension-facebook)
    local permissions = { "public_profile" }
    -- login using read permissions
    -- there is no need to specify a publishing audience when requesting read permissions
    facebook.login_with_permissions(permissions, facebook.AUDIENCE_NONE, function(self, data)
        local body = nakama.create_api_account_facebook(facebook.access_token())
        local result = nakama.authenticate_facebook(client, body, true, "mycustomusername")
        if not result.token then
            print("Unable to login")
            return
        end
        -- store the token and use it when communicating with the server
        nakama.set_bearer_token(client, result.token)
    end)
    ```

You can add a button to your UI to login with Facebook.

=== "Unity"
    ```csharp
    FB.Login("email", (ILoginResult result) => {
        if (FB.IsLoggedIn) {
            var oauthToken = Facebook.Unity.AccessToken.CurrentAccessToken.TokenString;
            var session = await client.LinkFacebookAsync(session, accesstoken);
            Debug.LogFormat("Session: '{0}'.", session.AuthToken);
        }
    });
    ```

#### Facebook Instant

Ensure that you've initialized the Facebook Instant Games SDK using `FBInstant.initializeAsync()`.

=== "JavaScript"
    ```js
    const result = await FBInstant.player.getSignedPlayerInfoAsync();
    var session = await client.authenticateFacebookInstantGame(result.getSignature());
    console.info("Successfully authenticated: %o", session);
    ```

!!! Note "Server configuration"
    Ensure you've [configured](../getting-started/configuration.md#facebook-instant-game) your FB Instant App secret for Nakama.


=== "Defold"
    ```lua
    -- Use the official Defold Facebook Instant Games integration (www.defold.com/extension-fbinstant)
    fbinstant.get_signed_player_info("developer payload", function(self, signature)
        local body = nakama.create_api_account_facebook_instant_game(signature)
        local result = nakama.authenticate_facebook_instant_game(client, body, true, "mycustomusername")
        if not result.token then
            print("Unable to login")
            return
        end
        -- store the token and use it when communicating with the server
        nakama.set_bearer_token(client, result.token)
    end)
    ```

#### Google

Similar to Facebook for register and login you should use one of Google's client SDKs.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/google?create=true&username=mycustomusername" \
      --user 'defaultkey:' \
      --data '{"token":"valid-oauth-token"}'
    ```

=== "JavaScript"
    ```js
    const playerIdToken = "...";
    const create = true;
    const session = await client.authenticateGoogle(oauthToken, create, "mycustomusername");
    console.info("Successfully authenticated: %o", session);
    ```

=== ".NET"
    ```csharp
    const string playerIdToken = "...";
    var session = await client.AuthenticateGoogleAsync(playerIdToken);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    const string playerIdToken = "...";
    var session = await client.AuthenticateGoogleAsync(playerIdToken);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
      CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string oauthToken = "...";
    client->authenticateGoogle(oauthToken, "mycustomusername", true, {}, successCallback, errorCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const oauthToken = "...";
    const create = true;
    client.authenticateGoogle(oauthToken, create, "mycustomusername")
      .then(function(session) {
          cc.log("Authenticated successfully. User ID:", session.user_id);
        },
        function(error) {
          cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string oauthToken = "...";
    client->authenticateGoogle(oauthToken, "mycustomusername", true, {}, successCallback, errorCallback);
    ```

=== "Java"
    ```java
    String playerIdToken = "...";
    Session session = client.authenticateGoogle(oauthToken).get();
    System.out.format("Session %s", session.getAuthToken());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let playerIdToken = "..."
    let message = AuthenticateMessage(google: oauthToken)
    client.register(with: message).then { session in
      NSLog("Session: %@", session.token)
    }.catch { err in
      NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    ```

=== "Godot"
    ```gdscript
    var oauth_token = "..."
    var session : NakamaSession = yield(client.authenticate_google_async(oauth_token), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/google?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
      "token": "...",
    }
    ```

=== "Defold"
    ```lua
    local body = nakama.create_api_account_google(oauth_token)
    local result = nakama.authenticate_google(client, body, true, "mycustomusername")
    if not result.token then
        print("Unable to login")
        return
    end
    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

#### Game Center

Apple devices have builtin authentication which can be done without user interaction through Game Center. The register or login process is a little complicated because of how Apple's services work.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/gamecenter?create=true&username=mycustomusername" \
      --user 'defaultkey:' \
      --data '{"player_id":"...", "bundle_id":"...", "timestamp_seconds":0, "salt":"...", "public_key_url":"..."}'
    ```

=== ".NET"
    ```csharp
    var bundleId = "...";
    var playerId = "...";
    var publicKeyUrl = "...";
    var salt = "...";
    var signature = "...";
    var timestamp = "...";
    var session = await client.AuthenticateGameCenterAsync(bundleId, playerId,
        publicKeyUrl, salt, signature, timestamp);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    // You'll need to use native code (Obj-C) with Unity.
    // The "UnityEngine.SocialPlatforms.GameCenter" doesn't give enough information
    // to enable authentication.

    // We recommend you use a library which handles native Game Center auth like
    // https://github.com/desertkun/GameCenterAuth

    var bundleId = "...";
    var playerId = "...";
    var publicKeyUrl = "...";
    var salt = "...";
    var signature = "...";
    var timestamp = "...";
    var session = await client.AuthenticateGameCenterAsync(bundleId, playerId,
        publicKeyUrl, salt, signature, timestamp);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
      CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    auto errorCallback = [](const NError& error)
    {
    };

    std::string playerId = "...";
    std::string    bundleId = "...";
    NTimestamp timestampSeconds = "...";
    std::string salt = "...";
    std::string signature = "...";
    std::string publicKeyUrl = "...";

    client->authenticateGameCenter(
      playerId,
      bundleId,
      timestampSeconds,
      salt,
      signature,
      publicKeyUrl,
      "mycustomusername",
      true,
      {},
      successCallback,
      errorCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const player_id = "...";
    const bundle_id = "...";
    const timestamp_seconds = "...";
    const salt = "...";
    const signature = "...";
    const public_key_url = "...";
    client.authenticateGameCenter({
      player_id: player_id,
      bundle_id: bundle_id,
      password: password,
      timestamp_seconds: timestamp_seconds,
      salt: salt,
      signature: signature,
      public_key_url: public_key_url,
      username: "mycustomusername",
      create: true
    }).then(function(session) {
          cc.log("Authenticated successfully. User ID:", session.user_id);
        },
        function(error) {
          cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
    };

    auto errorCallback = [](const NError& error)
    {
    };

    std::string playerId = "...";
    std::string    bundleId = "...";
    NTimestamp timestampSeconds = "...";
    std::string salt = "...";
    std::string signature = "...";
    std::string publicKeyUrl = "...";

    client->authenticateGameCenter(
      playerId,
      bundleId,
      timestampSeconds,
      salt,
      signature,
      publicKeyUrl,
      "mycustomusername",
      true,
      {},
      successCallback,
      errorCallback);
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let playerID : String = "..."
    let bundleID : String = "..."
    let base64salt : String = "..."
    let base64signature : String = "..."
    let publicKeyURL : String = "..."
    let timestamp : Int = 0

    let message = AuthenticateMessage(
        gamecenter: bundleID, playerID: playerID, publicKeyURL: publicKeyURL,
        salt: base64salt, timestamp: timestamp, signature: base64signature)
    client.register(with: message).then { session in
      NSLog("Session: %@", session.token)
    }.catch { err in
      NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    // Use client.login(...) after register.
    ```

=== "Godot"
    ```gdscript
    var bundle_id = "..."
    var player_id = "..."
    var public_key_url = "..."
    var salt = "..."
    var signature = "..."
    var timestamp = "..."
    var session : NakamaSession = yield(client.authenticate_game_center_async(bundle_id, player_id, public_key_url, salt, signature, timestamp), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/gamecenter?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
      "player_id": "...",
      "bundle_id": "...",
      "timestamp_seconds": 0,
      "salt": "...",
      "signature": "...",
      "public_key_url": "..."
    }
    ```

=== "Defold"
    ```lua
    -- Use https://defold.com/assets/gamekit/
    local bundle_id = "..."
    local player_id = "..."
    local public_key_url = "..."
    local salt = "..."
    local signature = "..."
    local timestamp_seconds = 0
    local body = nakama.create_api_account_game_center(bundle_id, player_id, public_key_url, salt, signature, timestamp_seconds)
    local result = nakama.authenticate_game_center(client, body, true, "mycustomusername")
    if not result.token then
        print("Unable to login")
        return
    end
    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

#### Steam

Steam requires you to configure the server before you can register a user.

!!! Note "Server configuration"
    Have a look at the [configuration](../getting-started/configuration.md) section for what settings you need for the server.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/steam?create=true&username=mycustomusername" \
      --user 'defaultkey' \
      --data '{"token":"valid-steam-token"}'
    ```

=== ".NET"
    ```csharp
    const string token = "...";
    var session = await client.AuthenticateSteamAsync(token);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    const string token = "...";
    var session = await client.AuthenticateSteamAsync(token);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
      CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string token = "...";
    string username = "mycustomusername";
    bool create = true;
    client->authenticateSteam(token, username, create, {}, successCallback, errorCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const token = "...";
    const create = true;
    client.authenticateSteam(token, create, "mycustomusername")
      .then(function(session) {
          cc.log("Authenticated successfully. User ID:", session.user_id);
        },
        function(error) {
          cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string token = "...";
    string username = "mycustomusername";
    bool create = true;
    client->authenticateSteam(token, username, create, {}, successCallback, errorCallback);
    ```

=== "Java"
    ```java
    String token = "...";
    Session session = client.authenticateSteam(token).get();
    System.out.format("Session %s", session.getAuthToken());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let sessionToken = "..."
    let message = AuthenticateMessage(steam: sessionToken)
    client.register(with: message).then { session in
      NSLog("Session: %@", session.token)
    }.catch { err in
      NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    // Use client.login(...) after register.
    ```

=== "Godot"
    ```gdscript
    var steam_token = "..."
    var session : NakamaSession = yield(client.authenticate_steam_async(steam_token), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/steam?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
      "token": "...",
    }
    ```

=== "Defold"
    ```lua
    -- Use https://defold.com/assets/steamworks/
    local body = nakama.create_api_account_steam(steam_token)
    local result = nakama.authenticate_steam(client, body, true, "mycustomusername")
    if not result.token then
        print("Unable to login")
        return
    end
    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

### Custom

A custom identifier can be used in a similar way to a device identifier to login or register a user. This option should be used if you have an external or custom user identity service which you want to use. For example EA's Origin service handles accounts which have their own user IDs.

A custom identifier must contain alphanumeric characters with dashes and be between 6 and 128 bytes.

You can choose a custom username when creating the account. To do this, set `username` to a custom name. If you want to only authenticate without implicitly creating a user account, set `create` to false.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/authenticate/custom?create=true&username=mycustomusername" \
      --user 'defaultkey:' \
      --data '{"id":"some-custom-id"}'
    ```

=== "JavaScript"
    ```js
    const customId = "some-custom-id";
    const create = true;
    const session = await client.authenticateCustom(customId, create, "mycustomusername");
    console.info("Successfully authenticated:", session);
    ```

=== ".NET"
    ```csharp
    const string customId = "some-custom-id";
    var session = await client.AuthenticateCustomAsync(customId);
    System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
    ```

=== "Unity"
    ```csharp
    const string customId = "some-custom-id";
    var session = await client.AuthenticateCustomAsync(customId);
    Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
      CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string id = "some-custom-id";
    string username = "mycustomusername";
    bool create = true;
    client->authenticateCustom(id, username, create, {}, successCallback, errorCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const customId = "some-custom-id";
    const create = true;
    client.authenticateCustom(customId, create, "mycustomusername")
      .then(function(session) {
          cc.log("Authenticated successfully. User ID:", session.user_id);
        },
        function(error) {
          cc.error("authenticate failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto successCallback = [](NSessionPtr session)
    {
        std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
    };

    auto errorCallback = [](const NError& error)
    {
    };

    string id = "some-custom-id";
    string username = "mycustomusername";
    bool create = true;
    client->authenticateCustom(id, username, create, {}, successCallback, errorCallback);
    ```

=== "Java"
    ```java
    String customId = "some-custom-id";
    Session session = client.authenticateCustom(customId).get();
    System.out.format("Session %s", session.getAuthToken());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let customID = "some-custom-id"

    let message = AuthenticateMessage(custom: customID)
    client.register(with: message).then { session in
      NSLog("Session: %@", session.token)
    }.catch { err in
      NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    // Use client.login(...) after register.
    ```

=== "Godot"
    ```gdscript
    var custom_id = "some-custom-id"
    var session : NakamaSession = yield(client.authenticate_custom_async(custom_id), "completed")
    if session.is_exception():
        print("An error occured: %s" % session)
        return
    print("Successfully authenticated: %s" % session)
    ```

=== "REST"
    ```
    POST /v2/account/authenticate/custom?create=true&username=mycustomusername
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Basic base64(ServerKey:)

    {
      "id": "some-custom-id",
    }
    ```

=== "Defold"
    ```lua
    local body = nakama.create_api_account_custom(custom_id)
    local result = nakama.authenticate_custom(client, body, true, "mycustomusername")
    if not result.token then
        print("Unable to login")
        return
    end
    -- store the token and use it when communicating with the server
    nakama.set_bearer_token(client, result.token)
    ```

## Session

When an authentication call succeeds, the server responds with a [session](/docs/session) object. The session object contains at least the following:

- The user ID
- The username
- A set of variables cached in the token.
- The creation time
- The expiration time

Once the client obtains the session object, you can utilize Nakama's realtime features such as [multiplayer](/docs/gameplay-multiplayer-realtime), [notifications](/docs/social-in-app-notifications) and [status updates](/docs/social-status), [passing stream data](/docs/advanced-streams) or [real-time chat](/docs/social-realtime-chat).

=== "JavaScript"
    ```js
    var socket = client.createSocket();
    session = await socket.connect(session);
    console.info("Socket connected.");
    ```

=== ".NET"
    ```csharp
    var socket = Socket.From(client);
    await socket.ConnectAsync(session);
    System.Console.WriteLine("Socket connected.");
    ```

=== "Unity"
    ```csharp
    var socket = client.NewSocket();
    await socket.ConnectAsync(session);
    Debug.Log("Socket connected.");
    ```

=== "Cocos2d-x C++"
    ```cpp
    #include "NakamaCocos2d/NWebSocket.h"

    int port = 7350; // different port to the main API port
    bool createStatus = true; // if the server should show the user as online to others.
    // define realtime client in your class as NRtClientPtr rtClient;
    rtClient = client->createRtClient(port, NRtTransportPtr(new NWebSocket()));
    // define listener in your class as NRtDefaultClientListener listener;
    listener.setConnectCallback([]()
    {
      CCLOG("Socket connected.");
    });
    rtClient->setListener(&listener);
    rtClient->connect(session, createStatus);
    ```

=== "Cocos2d-x JS"
    ```js
    const socket = client.createSocket();
    socket.connect(session)
      .then(
          function() {
            cc.log("Socket connected.");
          },
          function(error) {
            cc.error("connect failed:", JSON.stringify(error));
          }
        );
    ```

=== "C++"
    ```cpp
    int port = 7350; // different port to the main API port
    bool createStatus = true; // if the server should show the user as online to others.
    // define realtime client in your class as NRtClientPtr rtClient;
    rtClient = client->createRtClient(port);
    // define listener in your class as NRtDefaultClientListener listener;
    listener.setConnectCallback([]()
    {
      cout << "Socket connected." << endl;
    });
    rtClient->setListener(&listener);
    rtClient->connect(session, createStatus);
    ```

=== "Java"
    ```java
    SocketClient socket = client.createSocket();
    socket.connect(session, new AbstractSocketListener() {}).get();
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let session : Session = someSession // obtained from register or login.
    client.connect(with: session).then { _ in
      NSLog("Socket connected.")
    });
    ```

=== "Godot"
    ```gdscript
    # Make this a node variable, or it will disconnect when the function that creates it returns.
    onready var socket := Nakama.create_socket_from(client)

    func _ready():
        var connected : NakamaAsyncResult = yield(socket.connect_async(session), "completed")
        if connected.is_exception():
            print("An error occured: %s" % connected)
            return
        print("Socket connected.")
    ```

=== "Defold"
    ```lua
    local socket = nakama.create_socket(client)
    local ok, err = nakama.socket_connect(socket)
    if not ok then
        print("Unable to connect: ", err)
        return
    end
    ```

## Link or unlink

You can link one or more other login option to the current user. This makes it easy to support multiple logins with each user and easily identify a user across devices.

You can only link device Ids, custom Ids, and social provider IDs which are not already in-use with another user account.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/link/custom" \
      --header 'Authorization: Bearer $session' \
      --data '{"id":"some-custom-id"}'
    ```

=== "JavaScript"
    ```js
    const customId = "some-custom-id";
    const success = await client.linkCustom(session, customId);
    console.log("Successfully linked custom ID to current user.");
    ```

=== ".NET"
    ```csharp
    const string customId = "some-custom-id";
    await client.LinkCustomAsync(session, customId);
    System.Console.WriteLine("Id '{0}' linked for user '{1}'", customId, session.UserId);
    ```

=== "Unity"
    ```csharp
    const string customid = "some-custom-id";
    await client.LinkCustomAsync(session, customId);
    Debug.LogFormat("Id '{0}' linked for user '{1}'", customId, session.UserId);
    ```

=== "Cocos2d-x C++"
    ```cpp
    auto linkFailedCallback = [](const NError& error)
    {
    };

    auto linkSucceededCallback = []()
    {
      CCLOG("Linked successfully");
    };

    std::string customid = "some-custom-id";

    client->linkCustom(customid, linkSucceededCallback, linkFailedCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const customId = "some-custom-id";
    client.linkCustom(session, customId)
      .then(function() {
          cc.log("Linked successfully");
        },
        function(error) {
          cc.error("link failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto linkFailedCallback = [](const NError& error)
    {
    };

    auto linkSucceededCallback = []()
    {
      cout << "Linked successfully" << endl;
    };

    std::string customid = "some-custom-id";

    client->linkCustom(customid, linkSucceededCallback, linkFailedCallback);
    ```

=== "Java"
    ```java
    String customId = "some-custom-id";
    client.linkCustom(session, customId).get();
    System.out.format("Id %s linked for user %s", customId, session.getUserId());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let id = "some-custom-id"
    var message = SelfLinkMessage(device: id);
    client.send(with: message).then {
      NSLog("Successfully linked device ID to current user.")
    }.catch { err in
      NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    ```

=== "Godot"
    ```gdscript
    var custom_id = "some-custom-id"
    var linked : NakamaAsyncResult = yield(client.link_custom_async(session, custom_id), "completed")
    if linked.is_exception():
        print("An error occured: %s" % linked)
        return
    print("Id '%s' linked for user '%s'" % [custom_id, session.user_id])
    ```

=== "REST"
    ```
    POST /v2/account/link/custom
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Bearer <session token>

    {
        "id":"some-custom-id"
    }
    ```

=== "Defold"
    ```lua
    local body = nakama.create_api_account_custom(custom_id)
    local result = nakama.link_custom(client, body)
    if result.error then
        print("An error occurred:", result.error)
        return
    end
    ```

You can unlink any linked login options for the current user.

=== "cURL"
    ```sh
    curl "http://127.0.0.1:7350/v2/account/unlink/custom" \
      --header 'Authorization: Bearer $session' \
      --data '{"id":"some-custom-id"}'
    ```

=== "JavaScript"
    ```js
    const customId = "some-custom-id";
    const success = await client.unlinkCustom(session, customId);
    console.info("Successfully unlinked custom ID from the current user.");
    ```

=== ".NET"
    ```csharp
    const string customId = "some-custom-id";
    await client.UnlinkCustomAsync(session, customId);
    System.Console.WriteLine("Id '{0}' unlinked for user '{1}'", customId, session.UserId);
    ```

=== "Unity"
    ```csharp
    const string customId = "some-custom-id";
    await client.UnlinkCustomAsync(session, customId);
    Debug.LogFormat("Id '{0}' unlinked for user '{1}'", customId, session.UserId);
    ```


=== "Cocos2d-x C++"
    ```cpp
    auto unlinkFailedCallback = [](const NError& error)
    {
    };

    auto unlinkSucceededCallback = []()
    {
      CCLOG("Successfully unlinked custom ID from the current user.");
    };

    std::string customid = "some-custom-id";

    client->unlinkCustom(customid, unlinkSucceededCallback, unlinkFailedCallback);
    ```

=== "Cocos2d-x JS"
    ```js
    const customId = "some-custom-id";
    client.unlinkCustom(session, customId)
      .then(function() {
          cc.log("Successfully unlinked custom ID from the current user.");
        },
        function(error) {
          cc.error("unlink failed:", JSON.stringify(error));
        });
    ```

=== "C++"
    ```cpp
    auto unlinkFailedCallback = [](const NError& error)
    {
    };

    auto unlinkSucceededCallback = []()
    {
      cout << "Successfully unlinked custom ID from the current user." << endl;
    };

    std::string customid = "some-custom-id";

    client->unlinkCustom(customid, unlinkSucceededCallback, unlinkFailedCallback);
    ```

=== "Java"
    ```java
    String customId = "some-custom-id";
    client.unlinkCustom(session, customId).get();
    System.out.format("Id %s unlinked for user %s", customId, session.getUserId());
    ```

=== "Swift"
    ```swift
    // Requires Nakama 1.x
    let id = "some-custom-id"
    var message = SelfUnlinkMessage(device: id);
    client.send(with: message).then {
      NSLog("Successfully unlinked device ID from current user.")
    }.catch { err in
      NSLog("Error %@ : %@", err, (err as! NakamaError).message)
    }
    ```

=== "Godot"
    ```gdscript
    var custom_id = "some-custom-id"
    var unlinked : NakamaAsyncResult = yield(client.unlink_custom_async(session, custom_id), "completed")
    if unlinked.is_exception():
        print("An error occured: %s" % unlinked)
        return
    print("Id '%s' unlinked for user '%s'" % [custom_id, session.user_id])
    ```

=== "REST"
    ```
    POST /v2/account/unlink/custom
    Host: 127.0.0.1:7350
    Accept: application/json
    Content-Type: application/json
    Authorization: Bearer <session token>

    {
      "id":"some-custom-id"
    }
    ```

=== "Defold"
    ```lua
    local body = nakama.create_api_account_custom(custom_id)
    local result = nakama.unlink_custom(client, body)
    if result.error then
        print("An error occurred:", result.error)
        return
    end
    ```

You can link or unlink many different account options.

| Link | Description |
| ---- | ----------- |
| Apple | An Apple account. |
| Custom | A custom identifier from another identity service. |
| Device | A unique identifier for a device which belongs to the user. |
| Email | An email and password set by the user. |
| Facebook | A Facebook social account. You can optionally import Facebook Friends upon linking. |
| Game Center | An account from Apple's Game Center service. |
| Google | A Google account. |
| Steam | An account from the Steam network. |
