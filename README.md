![react-native-auth0](https://cdn.auth0.com/website/sdks/banners/react-native-auth0-banner.png)

[![Build Status][circleci-image]][circleci-url]
[![NPM version][npm-image]][npm-url]
[![Coverage][codecov-image]][codecov-url]
[![License][license-image]][license-url]
[![Downloads][downloads-image]][downloads-url]
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fauth0%2Freact-native-auth0.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fauth0%2Freact-native-auth0?ref=badge_shield)

📚 [Documentation](#documentation) • 🚀 [Getting Started](#getting-started) • ⏭️ [Next Steps](#next-steps) • ❓ [FAQs](https://github.com/auth0/react-native-auth0/blob/master/FAQ.md) • ❓ [Feedback](#feedback)

### ⚠️ Important Migration Notice: v4.0.0

We're excited to announce the release of react-native-auth0 `v4.0.0`! Please note that this update includes breaking changes that require your attention. To ensure a smooth transition, please review our
👉 [Migration Guide](https://github.com/auth0/react-native-auth0/blob/master/MIGRATION_GUIDE.md) 👈 for detailed instructions on updating your integration.

## Documentation

- [Quickstart](https://auth0.com/docs/quickstart/native/react-native/interactive)
- [Expo Quickstart](https://auth0.com/docs/quickstart/native/react-native-expo/interactive)
- [Sample App](https://github.com/auth0-samples/auth0-react-native-sample/tree/master/00-Login-Hooks)
- [Expo Sample App](https://github.com/auth0-samples/auth0-react-native-sample/tree/master/00-Login-Expo)
- [FAQs](https://github.com/auth0/react-native-auth0/blob/master/FAQ.md)
- [Examples](https://github.com/auth0/react-native-auth0/blob/master/EXAMPLES.md)
- [Docs Site](https://auth0.github.io/react-native-auth0/)

## Getting Started

### Requirements

This SDK targets apps that are using React Native SDK version `0.65.0` and up. If you're using an older React Native version, see the compatibility matrix below.

### Platform compatibility

The following shows platform minimums for running projects with this SDK:

| Platform | Minimum version |
| -------- | :-------------: |
| iOS      |      13.0       |
| Android  |       34        |

Our SDK requires a minimum iOS deployment target of 13.0. In your project's ios/Podfile, ensure your platform target is set to 13.0.

```
platform :ios, '13.0'
```

### Installation

First install the native library module:

### With [npm](https://www.npmjs.com)

`$ npm install react-native-auth0 --save`

### With [Yarn](https://yarnpkg.com/en/)

`$ yarn add react-native-auth0`

Then, you need to run the following command to install the ios app pods with Cocoapods. That will auto-link the iOS library:

`$ cd ios && pod install`

### Configure the SDK

You need to make your Android, iOS or Expo applications aware that an authentication result will be received from the browser. This SDK makes use of the Android's Package Name and its analogous iOS's Product Bundle Identifier to generate the redirect URL. Each platform has its own set of instructions.

#### Android

> Before version 2.9.0, this SDK required you to add an intent filter to the Activity on which you're going to receive the authentication result, and to use the `singleTask` **launchMode** in that activity. To migrate your app to version 2.9.0+, **remove both** and continue with the instructions below.
> You can also check out a sample migration diff [here](https://github.com/auth0-samples/auth0-react-native-sample/commit/69f79c83ceed40f44b239bbd16e79ecaa70ef70a).

Open your app's `build.gradle` file (typically at `android/app/build.gradle`) and add the following manifest placeholders:

```groovy
android {
    defaultConfig {
        // Add the next line
        manifestPlaceholders = [auth0Domain: "YOUR_AUTH0_DOMAIN", auth0Scheme: "${applicationId}.auth0"]
    }
    ...
}
```

The `auth0Domain` value must be replaced with your Auth0 domain value. So if you have `samples.us.auth0.com` as your Auth0 domain you would have a configuration like the following:

```groovy
android {
    defaultConfig {
        manifestPlaceholders = [auth0Domain: "samples.us.auth0.com", auth0Scheme: "${applicationId}.auth0"]
    }
    ...
}
```

The `applicationId` value will be auto-replaced at runtime with the package name or ID of your application (e.g. `com.example.app`). You can change this value from the `build.gradle` file. You can also check it at the top of your `AndroidManifest.xml` file.

> Note that if your Android application is using [product flavors](https://developer.android.com/studio/build/build-variants#product-flavors), you might need to specify different manifest placeholders for each flavor.

If you use a value other than `applicationId` in `auth0Scheme` you will also need to pass it as the `customScheme` option parameter of the `authorize` and `clearSession` methods.

Take note of this value as you'll be requiring it to define the callback URLs below.

> For more info please read the [React Native docs](https://facebook.github.io/react-native/docs/linking.html).

##### Skipping the Web Authentication setup

If you don't plan to use Web Authentication, you will notice that the compiler will still prompt you to provide the `manifestPlaceholders` values, since the `RedirectActivity` included in this library will require them, and the Gradle tasks won't be able to run without them.

Re-declare the activity manually with `tools:node="remove"` in your app's Android Manifest in order to make the manifest merger remove it from the final manifest file. Additionally, one more unused activity can be removed from the final APK by using the same process. A complete snippet to achieve this is:

```xml
<activity
    android:name="com.auth0.android.provider.AuthenticationActivity"
    tools:node="remove"/>
<!-- Optional: Remove RedirectActivity -->
<activity
    android:name="com.auth0.android.provider.RedirectActivity"
    tools:node="remove"/>
```

#### iOS

Inside the `ios` folder find the file `AppDelegate.[swift|m]` add the following to it:

```objc
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options
{
  return [RCTLinkingManager application:app openURL:url options:options];
}
```

Inside the `ios` folder open the `Info.plist` and locate the value for `CFBundleIdentifier`, e.g.

```xml
<key>CFBundleIdentifier</key>
<string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
```

and then below it register a URL type entry using the value of `CFBundleIdentifier` as the value for `CFBundleURLSchemes`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>None</string>
        <key>CFBundleURLName</key>
        <string>auth0</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>$(PRODUCT_BUNDLE_IDENTIFIER).auth0</string>
        </array>
    </dict>
</array>
```

If your application is generated using the React Native CLI, the default value of `$(PRODUCT_BUNDLE_IDENTIFIER)` matches `org.reactjs.native.example.$(PRODUCT_NAME:rfc1034identifier)`. Take note of this value as you'll be requiring it to define the callback URLs below. If desired, you can change its value using XCode in the following way:

- Open the `ios/TestApp.xcodeproj` file replacing 'TestApp' with the name of your app or run `xed ios` from a Terminal.
- Open your project's or desired target's **Build Settings** tab and on the search bar at the right type "Product Bundle Identifier".
- Replace the **Product Bundle Identifier** value with your desired application's bundle identifier name (e.g. `com.example.app`).
- If you've changed the project wide settings, make sure the same were applied to each of the targets your app has.

If you use a value other than `$(PRODUCT_BUNDLE_IDENTIFIER)` in the `CFBundleURLSchemes` field of the `Info.plist` you will also need to pass it as the `customScheme` option parameter of the `authorize` and `clearSession` methods.

> For more info please read the [React Native docs](https://facebook.github.io/react-native/docs/linking.html).

#### Expo

> :warning: This SDK is not compatible with "Expo Go" app because of custom native code. It is compatible with Custom Dev Client and EAS builds

To use the SDK with Expo, configure the app at build time by providing the `domain` and the `customScheme` values through the [Config Plugin](https://docs.expo.dev/guides/config-plugins/). To do this, add the following snippet to _app.json_ or _app.config.js_:

```json
{
  "expo": {
    ...
    "plugins": [
      [
        "react-native-auth0",
        {
          "domain": "YOUR_AUTH0_DOMAIN",
          "customScheme": "YOUR_CUSTOM_SCHEME"
        }
      ]
    ]
  }
}
```

> :info: If you want to switch between multiple domains in your app, refer [here](https://github.com/auth0/react-native-auth0/blob/master/EXAMPLES.md#domain-switching)

| API          | Description                                                                                                                                                                                                                                                                  |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| domain       | Mandatory: Provide the Auth0 domain that can be found at the [Application Settings](https://manage.auth0.com/#/applications)                                                                                                                                                 |
| customScheme | Optional: Custom scheme to build the callback URL with. The value provided here should be passed to the `customScheme` option parameter of the `authorize` and `clearSession` methods. The custom scheme should be a unique, all lowercase value with no special characters. |

Now you can run the application using `expo run:android` or `expo run:ios`.

### Callback URL(s)

Callback URLs are the URLs that Auth0 invokes after the authentication process. Auth0 routes your application back to this URL and appends additional parameters to it, including a token. Since callback URLs can be manipulated, you will need to add this URL to your Application's **Allowed Callback URLs** for security. This will enable Auth0 to recognize these URLs as valid. If omitted, authentication will not be successful.

On the Android platform this URL is case-sensitive. Because of that, this SDK will auto convert the Bundle Identifier (iOS) and Application ID (Android) values to lowercase in order to build the Callback URL with them. If any of these values contains uppercase characters a warning message will be printed in the console. Make sure to check that the right Callback URL is whitelisted in the Auth0 dashboard or the browser will not route successfully back to your application.

Go to the [Auth0 Dashboard](https://manage.auth0.com/#/applications), select your application and make sure that **Allowed Callback URLs** contains the URLs defined below.

If in addition you plan to use the log out method, you must also add these URLs to the **Allowed Logout URLs**.

> [!NOTE]
> Whenever possible, Auth0 recommends using [Android App Links](https://developer.android.com/training/app-links) and [Apple Universal Links](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content) for your callback and logout URLs. Custom URL schemes can be subject to [client impersonation attacks](https://datatracker.ietf.org/doc/html/rfc8252#section-8.6).
>
> 💡 If your Android app is using [product flavors](https://developer.android.com/studio/build/build-variants#product-flavors), you might need to specify different manifest placeholders for each flavor.

#### Android

##### Custom Scheme

```text
{YOUR_APP_PACKAGE_NAME}.auth0://{YOUR_AUTH0_DOMAIN}/android/{YOUR_APP_PACKAGE_NAME}/callback
```

##### App Link (Recommended):

```text
https://{YOUR_AUTH0_DOMAIN}/android/{YOUR_APP_PACKAGE_NAME}/callback
```

> Replace {YOUR_APP_PACKAGE_NAME} and {YOUR_AUTH0_DOMAIN} with your actual application package name and Auth0 domain. Ensure that {YOUR_APP_PACKAGE_NAME} is all lowercase.

To enable App Links, set the `auth0Scheme` to `https` in your `build.gradle` file.

```text
android {
    defaultConfig {
        manifestPlaceholders = [auth0Domain: "@string/com_auth0_domain", auth0Scheme: "https"]
    }
}
```

This configuration ensures that your app uses https for the callback URL scheme, which is required for Android App Links.

#### Enable Android App Links Support

[Android App Links](https://developer.android.com/training/app-links) allow an application to designate itself as the default handler of a given type of link. For example, clicking a URL in an email would open the link in the designated application. This guide will show you how to enable Android App links support for your Auth0-registered application using Auth0's Dashboard.

1.  Go to [Auth0 Dashboard > Applications > Applications](https://manage.auth0.com/#/applications), and select the name of the application to view.

2.  Scroll to the bottom of the Settings page, and select **Show Advanced Settings**.
3.  Select Device Settings, provide the [App Package Name and](https://developer.android.com/studio/build/application-id) the SHA256 fingerprints of your app’s signing certificate for your Android application, and select Save Changes.
    ![android-app-link](assets/android-app-link.png)

> You can use the following command to generate the fingerprint using the Java keytool in your terminal: `keytool -list -v -keystore my-release-key.keystore`

To learn more about signing certificates, see Android's [Sign Your App](https://developer.android.com/studio/publish/app-signing.html) developer documentation.

#### iOS

##### Custom Scheme

```text
{PRODUCT_BUNDLE_IDENTIFIER}.auth0://{YOUR_AUTH0_DOMAIN}/ios/{PRODUCT_BUNDLE_IDENTIFIER}/callback
```

##### Universal Link (Recommended):

```text
https://{YOUR_AUTH0_DOMAIN}/ios/{PRODUCT_BUNDLE_IDENTIFIER}/callback
```

> Replace `{PRODUCT_BUNDLE_IDENTIFIER}` and `{YOUR_AUTH0_DOMAIN}` with your actual product bundle identifier and Auth0 domain. Ensure that {PRODUCT_BUNDLE_IDENTIFIER} is all lowercase.

#### Configure an associated domain for iOS

> [!IMPORTANT]
> This step requires a paid Apple Developer account. It is needed to use Universal Links as callback and logout URLs.
> Skip this step to use a custom URL scheme instead.

##### Configure the Team ID and bundle identifier

Scroll to the end of the settings page of your Auth0 application and open **Advanced Settings > Device Settings**. In the **iOS** section, set **Team ID** to your [Apple Team ID](https://developer.apple.com/help/account/manage-your-team/locate-your-team-id/), and **App ID** to your app's bundle identifier.

![Screenshot of the iOS section inside the Auth0 application settings page](https://github.com/auth0/Auth0.swift/assets/5055789/7eb5f6a2-7cc7-4c70-acf3-633fd72dc506)

This will add your app to your Auth0 tenant's `apple-app-site-association` file.

##### Add the associated domain capability

In Xcode, go to the **Signing and Capabilities** [tab](https://developer.apple.com/documentation/xcode/adding-capabilities-to-your-app#Add-a-capability) of your app's target settings, and press the **+ Capability** button. Then select **Associated Domains**.

![Screenshot of the capabilities library inside Xcode](https://github.com/auth0/Auth0.swift/assets/5055789/3f7b0a70-c36c-46bf-9441-29f98724204a)

Next, add the following [entry](https://developer.apple.com/documentation/xcode/configuring-an-associated-domain#Define-a-service-and-its-associated-domain) under **Associated Domains**:

```text
webcredentials:YOUR_AUTH0_DOMAIN
```

<details>
  <summary>Example</summary>

If your Auth0 Domain were `example.us.auth0.com`, then this value would be:

```text
webcredentials:example.us.auth0.com
```

</details>

If you have a [custom domain](https://auth0.com/docs/customize/custom-domains), replace `YOUR_AUTH0_DOMAIN` with your custom domain.

> [!NOTE]
> For the associated domain to work, your app must be signed with your team certificate **even when building for the iOS simulator**. Make sure you are using the Apple Team whose Team ID is configured in the settings page of your Auth0 application.

Refer to the example of [Using custom scheme for web authentication redirection](https://github.com/auth0/react-native-auth0/blob/master/EXAMPLES.md#using-custom-scheme-for-web-authentication-redirection)

## Next Steps

> This SDK is OIDC compliant. To ensure OIDC compliant responses from the Auth0 servers enable the **OIDC Conformant** switch in your Auth0 dashboard under `Application / Settings / Advanced OAuth`. For more information please check [this documentation](https://auth0.com/docs/api-auth/intro#how-to-use-the-new-flows).

### Web Authentication

The SDK exports a React hook as the primary interface for performing [web authentication](#web-authentication) through the browser using Auth0 [Universal Login](https://auth0.com/docs/authenticate/login/auth0-universal-login).

Use the methods from the `useAuth0` hook to implement login, logout, and to retrieve details about the authenticated user.

See the [API Documentation](https://auth0.github.io/react-native-auth0/functions/useAuth0.html) for full details on the `useAuth0` hook.

First, import the `Auth0Provider` component and wrap it around your application. Provide the `domain` and `clientId` values as given to you when setting up your Auth0 app in the dashboard:

```js
import { Auth0Provider } from 'react-native-auth0';

const App = () => {
  return (
    <Auth0Provider domain="YOUR_AUTH0_DOMAIN" clientId="YOUR_AUTH0_CLIENT_ID">
      {/* YOUR APP */}
    </Auth0Provider>
  );
};

export default App;
```

You can also pass custom headers that will be included in all API requests:

```js
import { Auth0Provider } from 'react-native-auth0';

const App = () => {
  return (
    <Auth0Provider 
      domain="YOUR_AUTH0_DOMAIN" 
      clientId="YOUR_AUTH0_CLIENT_ID"
      headers={{ 'X-Custom-Header': 'custom-value' }}
    >
      {/* YOUR APP */}
    </Auth0Provider>
  );
};

export default App;
```

<details>
  <summary>Using the `Auth0` class</summary>

If you're not using React Hooks, you can simply instantiate the `Auth0` class:

```js
import Auth0 from 'react-native-auth0';

const auth0 = new Auth0({
  domain: 'YOUR_AUTH0_DOMAIN',
  clientId: 'YOUR_AUTH0_CLIENT_ID',
});
```

You can also pass custom headers that will be included in all API requests:

```js
import Auth0 from 'react-native-auth0';

const auth0 = new Auth0({
  domain: 'YOUR_AUTH0_DOMAIN',
  clientId: 'YOUR_AUTH0_CLIENT_ID',
  headers: {
    'X-Custom-Header': 'custom-value',
  }
});
```
</details>

Then import the hook into a component where you want to get access to the properties and methods for integrating with Auth0:

```js
import { useAuth0 } from 'react-native-auth0';
```

#### Login

Use the `authorize` method to redirect the user to the Auth0 [Universal Login](https://auth0.com/docs/authenticate/login/auth0-universal-login) page for authentication. If `scope` is not specified, `openid profile email` is used by default.

- The `isLoading` property is set to true once the authentication state of the user is known to the SDK.
- The `user` property is populated with details about the authenticated user. If `user` is `null`, no user is currently authenticated.
- The `error` property is populated if any error occurs.

```js
const Component = () => {
  const { authorize, user, isLoading, error } = useAuth0();

  const login = async () => {
    await authorize();
  };

  if (isLoading) {
    return (
      <View>
        <Text>SDK is Loading</Text>
      </View>
    );
  }

  return (
    <View>
      {!user && <Button onPress={login} title="Log in" />}
      {user && <Text>Logged in as {user.name}</Text>}
      {error && <Text>{error.message}</Text>}
    </View>
  );
};
```

<details>
  <summary>Using the `Auth0` class</summary>
  
  ```js
  auth0.webAuth
    .authorize()
    .then(credentials => console.log(credentials))
    .catch(error => console.log(error));
  ```
</details>

> Web Authentication flows require a Browser application installed on the device. When no Browser is available, an error of type `a0.browser_not_available` will be raised via the provided callback.

##### SSO Alert Box (iOS)

![ios-sso-alert](assets/ios-sso-alert.png)

Check the [FAQ](FAQ.md) for more information about the alert box that pops up **by default** when using Web Auth on iOS.

> See also [this blog post](https://developer.okta.com/blog/2022/01/13/mobile-sso) for a detailed overview of Single Sign-On (SSO) on iOS.

#### Logout

Log the user out by using the `clearSession` method from the `useAuth0` hook.

```js
const Component = () => {
  const { clearSession, user } = useAuth0();

  const logout = async () => {
    await clearSession();
  };

  return <View>{user && <Button onPress={logout} title="Log out" />}</View>;
};
```

<details>
  <summary>Using the `Auth0` class</summary>

```js
auth0.webAuth.clearSession().catch((error) => console.log(error));
```

</details>

### Credentials Manager

- [Check for stored credentials](#check-for-stored-credentials)
- [Retrieve stored credentials](#retrieve-stored-credentials)
- [Local authentication](#local-authentication)
- [Credentials Manager errors](#credentials-manager-errors)

The Credentials Manager allows you to securely store and retrieve the user's credentials. The credentials will be stored encrypted in Shared Preferences on Android, and in the Keychain on iOS.

The `Auth0` class exposes the `credentialsManager` property for you to interact with using the API below.

> 💡 If you're using Web Auth (`authorize`) through Hooks, you do not need to manually store the credentials after login and delete them after logout; the SDK does this automatically.

#### Check for stored credentials

When the users open your app, check for valid credentials. If they exist, you can retrieve them and redirect the users to the app's main flow without any additional login steps.

```js
const isLoggedIn = await auth0.credentialsManager.hasValidCredentials();

if (isLoggedIn) {
  // Retrieve credentials and redirect to the main flow
} else {
  // Redirect to the login page
}
```

#### Retrieve stored credentials

The credentials will be automatically renewed using the [refresh token](https://auth0.com/docs/secure/tokens/refresh-tokens), if the access token has expired. **This method is thread safe**.

```js
const credentials = await auth0.credentialsManager.getCredentials();
```


> 💡 You do not need to call credentialsManager.saveCredentials() afterward. The Credentials Manager automatically persists the renewed credentials.

#### Requiring Authentication before obtaining Credentials

> :warning: The `requireLocalAuthentication` method is no longer available as part of the `CredentialsManager` class or the `useAuth0` Hook from v4 of the SDK.

> ℹ️ You need to use at least version `0.59.0` of React Native, as it uses `FragmentActivity` as the base activity, which is required for biometric authentication to work.

You can enable an additional level of user authentication before retrieving credentials using the local authentication supported by the device, for example PIN or fingerprint on Android, and Face ID or Touch ID on iOS.

Refer to the instructions below to understand how to enable authentication before retrieving credentials based on your setup:

**Using Auth0 Class:**

The `Auth0` class constructor now accepts a new parameter, which is an instance of the `LocalAuthenticationOptions` object. This needs to be passed while creating an instance of `Auth0` to enable authentication before obtaining credentials, as shown in the code snippet below:

```tsx
import Auth0 from 'react-native-auth0';
const localAuthOptions: LocalAuthenticationOptions = {
  title: 'Authenticate to retreive your credentials',
  subtitle: 'Please authenticate to continue',
  description: 'We need to authenticate you to retrieve your credentials',
  cancelTitle: 'Cancel',
  evaluationPolicy: LocalAuthenticationStrategy.deviceOwnerWithBiometrics,
  fallbackTitle: 'Use Passcode',
  authenticationLevel: LocalAuthenticationLevel.strong,
  deviceCredentialFallback: true,
};
const auth0 = new Auth0({
  domain: config.domain,
  clientId: config.clientId,
  localAuthenticationOptions: localAuthOptions,
});
```

**Using Hooks (Auth0Provider):**

`Auth0Provider` now accepts a new parameter, which is an instance of the `LocalAuthenticationOptions` object. This needs to be passed to enable authentication before obtaining credentials, as shown in the code snippet below:

```tsx
import { Auth0Provider } from 'react-native-auth0';

const localAuthOptions: LocalAuthenticationOptions = {
  title: 'Authenticate to retreive your credentials',
  subtitle: 'Please authenticate to continue',
  description: 'We need to authenticate you to retrieve your credentials',
  cancelTitle: 'Cancel',
  evaluationPolicy: LocalAuthenticationStrategy.deviceOwnerWithBiometrics,
  fallbackTitle: 'Use Passcode',
  authenticationLevel: LocalAuthenticationLevel.strong,
  deviceCredentialFallback: true,
};

const App = () => {
  return (
    <Auth0Provider
      domain={config.domain}
      clientId={config.clientId}
      localAuthenticationOptions={localAuthOptions}
    >
      {/* YOUR APP */}
    </Auth0Provider>
  );
};

export default App;
```

Detailed information on `LocalAuthenticationOptions` is available [here](#localauthenticationoptions)

**LocalAuthenticationOptions:**

The options for configuring the display of local authentication prompt, authentication level (Android only), and evaluation policy (iOS only).

**Properties:**

| Property                   | Type                                     | Description                                                                                                                                 | Applicable Platforms |
| -------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| `title`                    | `String`                                 | The title of the authentication prompt.                                                                                                     | Android, iOS         |
| `subtitle`                 | `String` (optional)                      | The subtitle of the authentication prompt.                                                                                                  | Android              |
| `description`              | `String` (optional)                      | The description of the authentication prompt.                                                                                               | Android              |
| `cancelTitle`              | `String` (optional)                      | The cancel button title of the authentication prompt.                                                                                       | Android, iOS         |
| `evaluationPolicy`         | `LocalAuthenticationStrategy` (optional) | The evaluation policy to use when prompting the user for authentication. Defaults to `deviceOwnerWithBiometrics`.                           | iOS                  |
| `fallbackTitle`            | `String` (optional)                      | The fallback button title of the authentication prompt.                                                                                     | iOS                  |
| `authenticationLevel`      | `LocalAuthenticationLevel` (optional)    | The authentication level to use when prompting the user for authentication. Defaults to `strong`.                                           | Android              |
| `deviceCredentialFallback` | `Boolean` (optional)                     | Should the user be given the option to authenticate with their device PIN, pattern, or password instead of a biometric. Defaults to `false` | Android              |

> :warning: You need a real device to test Local Authentication for iOS. Local Authentication is not available in simulators.

#### Credentials Manager errors

The Credentials Manager will only throw `CredentialsManagerError` exceptions. You can find more information in the details property of the exception.

```js
try {
  const credentials = await auth0.credentialsManager.getCredentials();
} catch (error) {
  console.log(error);
}
```

**Platform agnostic errors:**

You can access the platform agnostic generic error codes as below :

```js
try {
  const credentials = await auth0.credentialsManager.getCredentials();
} catch (error) {
  console.log(e.type);
}
```

_Note_ : We have platform agnostic error codes available only for `CredentialsManagerError` as of now.

| Generic Error Code    | Corresponding Error Code in Android                                                                                                                                                                                                                                                                                                              | Corresponding Error Code in iOS |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------- |
| `INVALID_CREDENTIALS` | `INVALID_CREDENTIALS`                                                                                                                                                                                                                                                                                                                            |                                 |
| `NO_CREDENTIALS`      | `NO_CREDENTIALS`                                                                                                                                                                                                                                                                                                                                 | `noCredentials`                 |
| `NO_REFRESH_TOKEN`    | `NO_REFRESH_TOKEN`                                                                                                                                                                                                                                                                                                                               | `noRefreshToken`                |
| `RENEW_FAILED`        | `RENEW_FAILED`                                                                                                                                                                                                                                                                                                                                   | `renewFailed`                   |
| `STORE_FAILED`        | `STORE_FAILED`                                                                                                                                                                                                                                                                                                                                   | `storeFailed`                   |
| `REVOKE_FAILED`       | `REVOKE_FAILED`                                                                                                                                                                                                                                                                                                                                  | `revokeFailed`                  |
| `LARGE_MIN_TTL`       | `LARGE_MIN_TTL`                                                                                                                                                                                                                                                                                                                                  | `largeMinTTL`                   |
| `INCOMPATIBLE_DEVICE` | `INCOMPATIBLE_DEVICE`                                                                                                                                                                                                                                                                                                                            |                                 |
| `CRYPTO_EXCEPTION`    | `CRYPTO_EXCEPTION`                                                                                                                                                                                                                                                                                                                               |                                 |
| `BIOMETRICS_FAILED`   | OneOf <br>`BIOMETRIC_NO_ACTIVITY`,`BIOMETRIC_ERROR_STATUS_UNKNOWN`,`BIOMETRIC_ERROR_UNSUPPORTED`,<br>`BIOMETRIC_ERROR_HW_UNAVAILABLE`,`BIOMETRIC_ERROR_NONE_ENROLLED`,`BIOMETRIC_ERROR_NO_HARDWARE`,<br>`BIOMETRIC_ERROR_SECURITY_UPDATE_REQUIRED`,`BIOMETRIC_AUTHENTICATION_CHECK_FAILED`,<br>`BIOMETRIC_ERROR_DEVICE_CREDENTIAL_NOT_AVAILABLE` | `biometricsFailed`              |
| `NO_NETWORK`          | `NO_NETWORK`                                                                                                                                                                                                                                                                                                                                     |                                 |
| `API_ERROR`           | `API_ERROR`                                                                                                                                                                                                                                                                                                                                      |                                 |

## Feedback

### Contributing

We appreciate feedback and contribution to this repo! Before you get started, please see the following:

- [Auth0's general contribution guidelines](https://github.com/auth0/open-source-template/blob/master/GENERAL-CONTRIBUTING.md)
- [Auth0's code of conduct guidelines](https://github.com/auth0/open-source-template/blob/master/CODE-OF-CONDUCT.md)
- [This repo's development guide](DEVELOPMENT.md)

### Raise an issue

To provide feedback or report a bug, [please raise an issue on our issue tracker](https://github.com/auth0/react-native-auth0/issues).

### Vulnerability Reporting

Please do not report security vulnerabilities on the public Github issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

---

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://cdn.auth0.com/website/sdks/logos/auth0_light_mode.png" width="150">
    <source media="(prefers-color-scheme: dark)" srcset="https://cdn.auth0.com/website/sdks/logos/auth0_dark_mode.png" width="150">
    <img alt="Auth0 Logo" src="https://cdn.auth0.com/website/sdks/logos/auth0_light_mode.png" width="150">
  </picture>
</p>
<p align="center">Auth0 is an easy to implement, adaptable authentication and authorization platform. To learn more checkout <a href="https://auth0.com/why-auth0">Why Auth0?</a></p>
<p align="center">
This project is licensed under the MIT license. See the <a href="https://github.com/auth0/react-native-auth0/blob/master/LICENSE"> LICENSE</a> file for more info.</p>

<!-- Variables -->

[npm-image]: https://img.shields.io/npm/v/react-native-auth0.svg?style=flat-square
[npm-url]: https://npmjs.org/package/react-native-auth0
[circleci-image]: https://img.shields.io/circleci/project/github/auth0/react-native-auth0.svg?branch=master&style=flat-square
[circleci-url]: https://circleci.com/gh/auth0/react-native-auth0
[codecov-image]: https://img.shields.io/codecov/c/github/auth0/react-native-auth0.svg?style=flat-square
[codecov-url]: https://codecov.io/github/auth0/react-native-auth0
[license-image]: https://img.shields.io/npm/l/react-native-auth0.svg?style=flat-square
[license-url]: #license
[downloads-image]: https://img.shields.io/npm/dm/react-native-auth0.svg?style=flat-square
[downloads-url]: https://npmjs.org/package/react-native-auth0
