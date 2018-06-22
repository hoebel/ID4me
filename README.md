![id4me](http://id4me.org/wp-content/uploads/2018/03/ID4me_logo_2c_pos_rgb.png)

ID4me Open Prototype API
--------------------

In order to help interested parties better understand how the **ID4me** protocol works and give the developers a 
starting point for their implementation, we have built an **Open Prototype API** which includes the core functionalities used by 
a relying party. The **Open Prototype API** was built as a proof of concept and it is not intended to be used in production.   

The api works together with the **AUTHORITY** implementation provided by **DENIC** (https://auth.freedom-id.de/) and the **AGENT** implementation provided by **???** (https://identityagent.de/). 


Core functionalities
--------------------

The Open Prototype API implementation has any functionalities as they are needed to implement a **RELYING PARTY** presented in 
[ID4me Technical Documentation](http://id4me.org/files/ID4me_Technical_Overview_v1.2.2.pdf):

* new client registration (*user has to manually add the DNS records) 
* user authorization
* user authentication
* user info retrieval
* claims selection
 

Technical details
--------------------

The **Open Prototype API** is written in **Java** and is using the file system as storage for the registration data.
Our main concern was to build something simple to understand and easy to get it running on a local machine. 
But keep in mind that when the application is running on a local environment some of the functionalities will not be available as the authority will not be able to send requests to the agent.

How to use
--------------------

* Build jar file locally
* Include the jar file in your java classpath
* Create the configuration files required by the `Id4meLogon` constructor
* Define an id4me, which is the domain wich points to the ID4me logon configuration in the dns
```java
// The ID4me of the user, used to discover the ID4me dns record
String id4me = “user.id4me.org” 
```
* Create a new `logon_handler`
```java
Id4meLogon logon_handler = new Id4meLogon(
	"id4me.properties", "claims.parameters.json");
```

* Create new `session_data
```java
// The first parameter is just the id4me of the user, 
// the second parameter is indicating whether a dynamic client registration shall be done 
// if the client is not already registered at the identity authority
Id4meSessionData session_data = logon_handler.createSessionData(id4me, true);
```

* Get the authorization uri from the `logon_handler`
```java
// response is the HttpServletRessponse of the web request
String authorizationUri = logon_handler.authorize(session_data);
response.sendRedirect(authorizationUri);
```

* Assuming that we have access to the created logon_handler and session_data objects for this web session our authorization end point now can authenticate the user. Therefore do we have to get the value of the parameter “code” from the HttpServletRequest request.
```java
String code = reqest.getParameter("code");
logon_handler.authenticate(session_data, code);
```

* The unique identity of an user is a String consists of the iss + # + sub. 
You now can get it from the `session_data` 
```java
String identity = session_data.getIdentityHandle();
```
* If you need the userinfo, you now can get it from the `logon_handler`
```java
// first we call userinfo(session_data) to fetch the userinfo data from the identity agent
boolean ok = logon_handler.userinfo(session_data);
if (ok) {
    // now we can get the data from the session_data object
    JSONObject userinfo = session_data.getUserinfo();
}
```




