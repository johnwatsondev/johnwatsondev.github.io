---
title: Keep it Secret, Keep it Safe "Slide"
layout: post
guid: urn:uuid:c798ca55-e941-4865-90cc-339638aae7ee
comments: true
tags:
  - Android Dev Summit 2015
---

原文：[Keep it Secret, Keep it Safe (Android Dev Summit 2015)](https://www.youtube.com/watch?v=fcWVV0Hafuk&index=4&list=PLWz5rJ2EKKc_Tt7q77qwyKRgytF1RzRx8)  
整理：[JohnWatsonDev](http://www.johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 1. Outline

- High level what is TLS and why should we use it everywhere
- Using TLS on the client and checking your work
- TLS mistakes and misconfigurations
- Questions

### 2. TLS: Why and what is it?
The network is not to be trusted. This has always been true but is especially for mobile devices.

- Devices connect to many different networks in a given day
  * Coffee shop Wi-Fi, Cellular network, Home Wi-Fi, Work Wi-Fi, ...
  * Anyone can run an open Wi-Fi network
- Devices contains and transmit lots of personal and private information

#### (1) Sensitive traffic
Canonical examples of sensitive traffic

- Login for your banking app
- Credit card details
- Private personal photos
- Emails to your boss
- Sensitive web traffic
- Health information

#### (2) What if that doesn't include you?
"I just run Chad's Boring Food Blog, why do I care? It's no sensitive."

**You still should secure your traffic**

#### (3) Non-sensitive content
**All** traffic must be protected. Things an attacker can do with non-sensitive traffic:

- Modify non-sensitive traffic:
  + Inject exploits into content
  + Modify content
  + Replace images, adds, etc.
- Track and snoop

**Goal: The network cannot affect the security of your device or data**

#### (4) Enter TLS
- Transport Layer Security
  + Previously know as SSL (Secure Socket Layer)
  + Usable with any protocol (e.g. HTTPS is HTTP over TLS)
- Establishes an end-to-end channel between two peers which offers
  + Integrity - detect tampering
  + The hard part is knowing you're talking to the right peer. We'll come back to this later when we talk about pitfalls and mistakes.
- Safe by default
  + Modern platforms have safe defaults and it is just as simple as using TLS.

### 3. Using TLS

#### (1) Using HTTP? Switch to HTTPS.
If your application uses standard HTTP libraries it is as simple as replacing http:// with https://

```java
URL url = new URL("https://android.com");
URLConnection urlConnection = url.openConnection();
InputStream in = urlConnection.getInputStream();
```

#### (2) What if you implement your own protocol?
This is less common, but if you do implement your own protocol on top of Sockets you can also use TLS using SSLSocket.

**BUT BE CAREFUL! SSLSocket has a very dangerous pitfall**. You must check that the peer's SSL certificate matches the hostname you're connecting to. **SSLSocket does not do this for you**.

If you fail to do this your connection is not secure.

```java
SocketFactory sf = SSLSocketFactory.getDefault();
SSLSocket socket = (SSLSocket) sf.createSocket("android.com", 443);
HostnameVerifier hv = HttpsURLConnection.getDefaultHostnameVerifier();
SSLSession s = socket.getSession();

// Verify that the certificate hostname is for android.com
if (!hv.verify("android.com", s)) {
  throw new SSLHandshakeException("Exception android.com, found " + s.getPeerPrincipal());
}
// Use socket
```

#### (3) Using TLS - On the server
SSL Labs has a good best practices document on what TLS protocol to support, key sizes, ciphers, and other server side configurations.
https://www.ssllabs.com/projects/best-practices/index.html

Most cloud providers support TLS.

### 4. Checking your work

#### (1) Are you sure you're using TLS?
Hopefully you're convinced that you should use TLS to protect your connections.

Now that you want to do that, how do you ensure that **all** your traffic is sent over TLS? How do you prevent accidental cleartext?

#### (2) The 'obvious' case - Hardcoded URLs
```java
URL url = new URL("https://example.com");
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
```

What about redirects?  
http://developer.android.com/reference/java/net/HttpURLConnection.html (Response Handling) says "This implementation doesn't follow redirects from HTTPS to HTTP or vice versa."

#### (3) A common cause of issues
Often a lot of the URLs your application hits aren't hard coded in your application but instead provided by a server.

```java
URL url = getUrlFromServer(...);
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
```

What if the backend starts sending http:// URLs?

#### (4) What about third party code?
Most applications include third party libraries for things like ads and analytics.

Is your application's and user's data being sent by those services over secure connections?

#### (5) New features in Marshmallow to help you
In order to help you accurately and easily determine if your application is making cleartext traffic in Marshmallow we added two features.

1. [Strict mode cleartext detection](https://developer.android.com/reference/android/os/StrictMode.VmPolicy.Builder.html#detectCleartextNetwork()) to help you while testing.
2. [usesCleartextTraffic application manifest attribute](http://developer.android.com/guide/topics/manifest/application-element.html#usesCleartextTraffic) to block accidental regressions on user devices.

**Note: These are not limited to HTTP/HTTPS**

#### (6) Strict mode
```java
StrictMode.VmPolicy policy =
  new StrictMode.VmPolicy.Builder()
      .detectCleartextNetwork()
      .penaltyDeath()
      .build();
StrictMode.setVmPolicy(policy);
```

Uses packet inspection to catch **all** non-TLS connections from your app.  
Useful during app development to make sure that your traffic is going out over TLS.  
Can result in false-positives such as with HTTP proxies, protocols with STARTTLS logic, or any secure traffic that doesn't begin with a TLS Client Hello.

#### (7) usesCleartextTraffic
AndroidManifest.xml

```java
<application android:usesCleartextTraffic="false" />
```

Indicates if the application intends to use cleartext traffic.  
If the flag if false libraries and services should refuse to send cleartext traffic at runtime.  
**Best effort only!**

- Supported
  + HttpsUrlConnection
  + OkHttp, Apache HttpClient
  + DownloadManager, MediaPlayer
- Not supported
  + WebView

#### (8) Why block vs upgrade
If we silently upgraded connections (ie: HTTP to HTTPS rewrites) it might allow newer devices to avoid these kinds of issues.

However:

- Older devices would not be protected.
- Upgrade logic isn't always well defined for non-HTTP protocols.
  + Rewrites might not work, connections may still break.

It's better for insecure connections to fail on newer devices and generate bug reports to get the application fixed properly.

### 5. Mistakes and misconfigurations

#### (1) So now you're using TLS
How do you know that you're doing TLS correctly?

Android, and most platforms out there, have secure defaults for TLS. So long as you're not trying to customize security checking code you're good to go.

#### (2) Mistake and misconfigurations
While TLS on Android is secure by default it is possible for the developer to override the trust checking used on a connection.

While trust checking should only be overwritten with secure logic, that is unfortunately often not the case.

Throughout all of this you can refer to [security-ssl](developer.android.com/training/articles/security-ssl.html) for more details and the correct way to solve common problems include code samples.

#### (3) Why is it done insecurely?
Most of the insecure trust checking code tends to result from insecure code samples and advice on the internet that the developer finds and uses without properly understanding the code or what it is doing.

Let's look at some example problems, the correct code and where to find it, and finally the insecure code we commonly see as a solution and the risks it introduces.

#### (4) Trust checking in TLS in a nutshell
TLS trust verification has a lot of parts, but we're going to focus on two very critical parts that have seen the most mistakes and misconfigurations.

- Check that the certificate matches who you want to talk to
- Check that the certificate is trusted

Note that these are subtly different.

- A trusted certificate for evil.com must **not** be trusted for android.com
- Just because a certificate says it is for android.com doesn't mean it is trusted. Anyone can say they are android.com

#### (5) Talking to the right destination
It is critical to make sure that you are talking to the expected destination.  
You don't want to accidentally send your sensitive data to evil.com instead of your server.

This is done for you if you use any standard HTTP stack in Android.  
**If you are using raw SSLSocket APIs then it is NOT done**.

#### (6) The problem
For legacy server or servers dealing with legacy clients choosing what certificate to present for a connection can be difficult.

If a server hosts both example.com and example.org each having completely different certificates, which should the server present when a client connects?

This was solved in TLS with Server Name identifier extension but some legacy clients might not send that extension. How do you ensure those still connect if presented with the example.org certificate instead of the example.com certificate?

#### (7) The right way
For Android the SNI extension has been supported for HttpsURLConnection since 2.3*, but if you still need to support older clients

```java
HostnameVerifier hostnameVerifier = new HostnameVerifier(){
  @Override
  public boolean verify(String hostname, SSLSession session){
    HostnameVerifier hv = HttpsURLConnection.getDefaultHostnameVerifier();
    return hv.verify("example.com", session);
  }
};
URL url = new URL("https://example.org/");
HttpsURLConnection urlConnection = (HttpsURLConnection) url.openConnection();
urlConnection.setHostnameVerifier(hostnameVerifier);
// Do stuff with urlConnection.
```

#### (8) The wrong way
A common insecure code sample provided for solving this is:

```java
HttpsURLConnection.setDefaultHostnameVerifier(new HostnameVerifier(){
  public boolean verify(String hostname, SSLSession session){return true;}
});
```

**DON'T USE THIS!**  
This code is vulnerable to a Man in The Middle attack.

#### (9) How is it vulnerable
By not checking the hostname for a certificate this code will trust that a connection to evil.com is a secure connection to android.com.

Anyone can go buy a domain and get a valid certificate for that hostname, and present it for another website. That certificate however **must not** be valid for other websites.

#### (10) Establishing Trust
Before we get into the next common mistake we need a very high level understanding of checking that the peer is trusted in TLS.

- Typically the identity of a TLS peer is their certificate
  + You could hard code the expected certificate
  + But certificates change regularly, for example google.com's changes every three months.

#### (11) What is a certificate?
- Name (e.g. google.com)
- Public key (RSA, EC) used by TLS to secure the channel
- Issuer who signed this certificate (e.g. Verisign)
- and more...
- Issuer also has a certificate
  + Who was it issued by? Do we trust them?
      * And so on and so on...
  + This forms a chain of trust

#### (12) Turtles all the way down
In the end you need to have some trusted set of certificates you do trust.

Operating systems include a set of trusted certificates, but applications may bundle their own.

Why can you trust the device CAs?

- Strict requirements, audits, etc
- Certificate Transparency, pinning to detect misuse

#### (13) The right way
The correct way to handle this is to use custom trusted roots in your connection while still properly checking that the chain of trust is valid and all the other parts of trust verification.

The code for this is unfortunately a little too verbose for presenting in a slide, but you can find it here: [UnknownCa](http://developer.android.com/training/articles/security-ssl.html#UnknownCa)

#### (14) The wrong way
There are a lot of bad code samples around trust checking.

```java
SSLContext ctx = SSLContext.getInstance("TLS");
ctx.init(null, new TrustManager[] {
  new X509TrustManager(){
    public void checkClientTrusted(X509Certificate[] chain, String authType){}
    public void checkServerTrusted(X509Certificate[] chain, String authType){}
    public X509Certificate[] getAcceptedIssuers(){
      return new X509Certificate[]{};
    }
  }
}, null);
HttpsURLConnection.setDefaultSSLSocketFactory(ctx.getSocketFactory());
```

Unfortunately the X509TrustManager API is not very clear, so let's take a deeper look at what this does.

#### (15) X509TrustManager
public void checkServerTrusted(X509Certificate[] chain, String authType){}

From the [X509TrustManager documentation](http://developer.android.com/reference/javax/net/ssl/X509TrustManager.html):  
Throws CertificateException if the certificate chain can't be validated or isn't trusted.

So this means that **ANY** certificate is trusted.

#### (16) The wrong way
Again **DO NOT USE THIS CODE!**  
We've seen numerous apps include this wrapped in an `if(DEBUG)` check, however we've also seen applications forget to disable the debug flag before shipping.

#### (17) What you should do

- If you're not overriding the secure defaults for trust checking you don't have to do anything.
- But if you are...
  + Make sure you actually should be overriding the defaults.
  + Don't blindly take code samples off the internet for security sensitive code. Especially without understanding it!
  + Use developer.android.com first when looking for solutions to TLS issues.

#### (18) What we're doing
Additionally, the Android Security team is working to help detect and prevent these types of bugs

- Detection and notification for applications in Play with these issues
- Tools to help detect these issues in devices and applications
- Investigating APIs that follow the "easy to be secure, hard to be insecure" mantra to move away from the confusing and easy to get wrong APIs from the previous slides

#### (19) What we're doing: Vulnerability alerts and blocks
- We deliver vulnerability alerts via email and the Play Developer Console for issues mentioned in this talk -- and others as well.
- Make sure you check the warnings -- apps that don't get fixed may be blocked
- You can confirm your fixes by uploading the apk to Play Developer Console and checking back after ~5 hours
- Lots of vulnerable apps have been fixed. Thank you to developers who have taken action.

#### (20) nogotofail
Network testing tool for finding misconfigurations, bugs, and leaks in network traffic.

- Released November 2014 on github: github.com/google/nogotofail
- Detects some of the issues we've talked about in this talk, and a lot more
- Runs on the network so supports any devices, not just Android, without any required changes to the device and without making the device unusable
- Supports optional client attribution and notification for easy debugging
- Does require some more advanced setup, but the power and effectiveness is worth it
