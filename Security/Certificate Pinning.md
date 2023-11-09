Certificate pinning can help improve the security of an app by ensuring that the app only trusts a specific set of certificates, rather than all of the certificates in the system’s trust store. This can also help prevent man-in-the-middle attacks, where an attacker attempts to intercept and modify network traffic.

Android N (API/SDK level version 25) brought several new features and improvements, one of which is the [Network Security Config](https://developer.android.com/training/articles/security-config#CertificatePinning). This feature allows developers to specify a custom XML configuration file that enables certificate pinning for their app, even when using WebView.

One issue with certificate pinning is that `android:networkSecurityConfig` is not backward compatible with older versions of Android. Another issue is that it can be difficult to implement when using self-signed certificates.

One solution is to use an internally forked library [TrustKit for Android](https://github.com/infinum/TrustKit-Android/tree/feature/trust-anchors).

## Configuring a pinning policy

Deploying certificate pinning requires initializing TrustKit with a pinning policy (domains, pins, and additional settings).

Sample code below shows how to configure certificate pinning using the public SHA-256 key. Note the use of a backup pin, it ensures that the connectivity of your app is unaffected.

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
   <domain-config>
       <domain>www.yourdomain.com</domain>
       <pin-set>
           <pin digest="SHA-256">k3XnEYQCK79AtL9GYnT/nyhsabas03V+bhRQYHQbpXU=</pin>
           <!-- backup pin -->
           <pin digest="SHA-256">2kOi4HdYYsvTR1sTIR7RHwlf2SescTrpza9ZrWy7poQ=</pin>
       </pin-set>
       <!-- Do not enforce pinning validation -->
       <trustkit-config enforcePinning="false"/>
   </domain-config>
</network-security-config>
```

Here’s an example of a configuration when using a certificate. It is also possible to declare multiple certificates.

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
   <domain-config>
       <domain>www.yourdomain.com</domain>
         <trust-anchors>
           <certificates src="@raw/cert_for_your_domain"/>
         </trust-anchors>
       <!-- Do not enforce pinning validation -->
       <trustkit-config enforcePinning="false"/>
   </domain-config>
</network-security-config>
```

## Initializing TrustKit with the Pinning Policy

The path to the XML policy should then be specified in the manifest to enable it as the app's Network Security Configuration on Android N and above:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config">
        ...
    </application>
</manifest>
```

Once TrustKit has been initialized and the client or connection's `SSLSocketFactory` has been set, it will verify the server's certificate chain against the configured pinning policy whenever an HTTPS connection is initiated.

```kotlin
// Using the default path - res/xml/network_security_config.xml
TrustKit.initializeWithNetworkSecurityConfiguration(context)

val url = URL("https://www.yourdomain.com")
val serverHostname: String = url.host

// HttpsUrlConnection
val connection: HttpsURLConnection = url.openConnection() as HttpsURLConnection
connection.sslSocketFactory = TrustKit.getInstance().getSSLSocketFactory(serverHostname)

// OkHttp 3
val client = OkHttpClient.Builder()
   .sslSocketFactory(OkHttp3Helper.getSSLSocketFactory(), OkHttp3Helper.getTrustManager())
   .addInterceptor(OkHttp3Helper.getPinningInterceptor())
   .followRedirects(false)
   .followSslRedirects(false)
   .build()
```
