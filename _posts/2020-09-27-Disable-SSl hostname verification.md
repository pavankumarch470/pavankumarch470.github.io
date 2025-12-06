---
published: true
---
When we open an SSL connection in Java, the default implementation of the SSL protocol performs below validations to ensure the requested host is not fake:<br/>
1.It performs the SSL handshake.<br/>
2. During the handshake, it validates if the server certificate is available in the JVM's truststore(by default, cacerts is the JVM truststore).<br/>
3. Once the valid certificate is found in the JVM truststore during the handshake. The validation of the 	Hostname verification takes place based on the "**CN**" and "**SAN**" entries available in the Server 		certificate.<br/>

For Example, if the server certificate has the below as the "Subject Alternate Name(SAN)" entries:<br/>
```java
 SubjectAlternativeName[
      DNSName: inchVM.domain.com
      DNSName: inkumarVM.domain.com
      IPAddress: 127.0.0.1
      ]
 ```
<br/>
And if we try to open the connection for the "https://11.22.33.44:1234/" (where the IP address matches one of the DNS names available in the certificate)using the code below:<br/>
 ```java
  java.net.URL.openConnection(“https://….”))
```
<br/>
The SSL handshake process fails with the following error:<br/>
```
The SSL handshake process failes with the below error:Caused by: java.security.cert.CertificateException: No subject alternative names matching IP address 11.22.33.44 found
     at sun.security.util.HostnameChecker.matchIP(HostnameChecker.java:168)
     at sun.security.util.HostnameChecker.match(HostnameChecker.java:94)
     at sun.security.ssl.X509TrustManagerImpl.checkIdentity(X509TrustManagerImpl.java:455)
     at sun.security.ssl.X509TrustManagerImpl.checkIdentity(X509TrustManagerImpl.java:436)
     at sun.security.ssl.X509TrustManagerImpl.checkTrusted(X509TrustManagerImpl.java:200)
```
<br/>

To overcome the above error while performing the SSL handshake, we can follow one of the following approaches:<br/>
1. re-generate the server certificates with all the valid DSN names and the associated IP address as well.<br/>
2. Or we need to overwrite the "verify" function and set the "HostnamVerifier" to use the new function.<br/>
 
Below is the example code for overwriting the  default "verify" function:<br/>
```java
HostnameVerifier validatingHosts = new HostnameVerifier() {
    public boolean verify(String hostname, SSLSession session) {
        //user logic for validating the Hostnames  
        return true;
        }
    };
HttpsURLConnection.setDefaultHostnameVerifier(validatingHosts);
```
