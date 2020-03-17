[Charles](https://www.charlesproxy.com/) is an HTTP proxy / HTTP monitor / Reverse Proxy that enables a developer to view all of the HTTP and SSL / HTTPS traffic between their machine and the Internet. This includes requests, responses and the HTTP headers (which contain the cookies and caching information).

It is very helpful when we debug api calls. Let's show one use case. 

You are debugging a call which gives you the response model called User that looks like this: 

```kotlin
class User {

    @Json(name = "name")
    val name: String? = null
    
    @Json(name = "age")
    val age: String? = null
    
...
```

There is a restriction request which says that users under age of 18 cannot use the app. You have implemented the logic for restricting young user from entering the app and you want to test it out. But there is only one test user available and it has value of age parameter set to 45. 

This is the case where Charles becomes very handy. You can set a breakpoint to intercept this call. After Charles does the interception, you can manually change the value of model's parameter.


## Setup Charles on Android

There are six steps to follow in order to setup Charles.

1. Build the App in Debug flavor and install it on your device. In Charles go to Proxy > Proxy Settings > Mac OS X  and disable it if activated.

2. Connect the device to the same network as your laptop is on (not a network which blocks proxies).

3. Modify the the WiFi connection on your device to use a proxy. It differs from device to device but it can go something like this: Long press on your wifi connection > Change/Adjust wifi > Show advanced settings > Proxy > Manually
There you need to set yor proxy host IP address (which is your laptop IP address, on Charles - Help > Local IP address) and you need to set the port number (for example 8888)

4. In Charles go to Proxy > SSL Proxying Settings > Add your host `most-awesome-api-ever.com` and port *.*

![Set your host](/img/charles-set-up-host.png)

5. On the phone use Chrome to navigate to [This page](http://chls.pro/ssl) and install the certificate.

6. Traffic should now be shown in Charles. Put a filter to your project to get rid of all the other traffic from your phone. If Charles shows "Unknown" label for all the calls from target api you can right click on it and select "Enable SSL proxying"

![Adjust traffic](/img/charles-focus-and-enable-ssl.png)

## How to use breakpoints

Breakpoints are very useful feature in Charles. You can use them doing following steps:

Right click on a wanted call and then select "Breakpoints".

![Select breakpoints](/img/charles-breakpoints-select.png)

After that execute the call and wait for Charles to intercept it. It will also intercept request (You can adjust this in settings). Just click on execute and continue to response. When response is shown, select JSON Text and simply change wanted value. When finished, click execute. And it is simple as that!

![Debug breakpoints](/img/charles-breakpoints-debug.png)


