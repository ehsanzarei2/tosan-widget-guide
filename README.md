# tosan-widget-guide

Widgets are loaded in Iframe in Netway; So developers can use any programming language and platform to create their own widgets.

Pay attention to the following points when developing and installing your widget.

1. The types of widgets with their feuters are listed in the table below. developers need to choose one of  the types when registering their widgets with 'saveWidget' of yaghut.

   1. When you choose Full type, Netway will display your widget in a seperate page
   2. The security of widgets is the responsibility of the bank and need to be checked by the bank.

Widget's type | Widget's width | Widget's height 
------------- | -------------- | ---------------
SMALL | 300 px | 216 px
SMALL | 300 px | 445 px
WIDE | 630 px | 216 px
MEDIUM | 630 px | 445 px
FULL | 900 px | 400 px

2. get clientId and clientSecret from suppurt operation team of tosan in the specified bank.

3. To register a widget, you must call "saveWidget" service from yaghut with required inputs .
### schema of "saveWidget" service:
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://service.yaghut.modern.tosan.com/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:saveWidget>
         <!--Optional:-->
         <context>
            <!--Zero or more repetitions:-->
            <data>
               <!--Optional:-->
               <key>?</key>
               <!--Optional:-->
               <value>?</value>
            </data>
         </context>
         <!--Optional:-->
         <request>
            <!--Optional:-->
            <description>?</description>
            <icon>cid:90332131574</icon>
            <!--Optional:-->
            <id>?</id>

            <!--Optional:-->
            <clientId> It is the client id that received fom step 2  </clientId>

            <!--Reqired:-->
            <startURL>  is your widget url and Netway and Jetway uses this URL to call your widget. </startURL>

            <!--Reqired:-->
            <title>?</title>
            
            <!--Reqired:-->
            <type>PERMANENT</type>

            <!--Reqired:-->
            <viewType> described in step 1</viewType>
         </request>
      </ser:saveWidget>
   </soapenv:Body>
</soapenv:Envelope>
``` 
4. when you developing your widget, please follow tips below:
   1. Widgets are loaded in Iframe in Netway, so you must set the 'X-Frame-Options' Header value to 'allow-from' for the Netway's URL.
   2. To ensure the widget is available, Netway sends a HTTP HEAD method request to the URL which you fill in the 'URL Schema'.
   3. If the Widget responses with 200 Http status code,Netway will send a HTTP GET method request with 'ssoJwt' query parameter to the URL which fill in the 'URL Schema'.  
   4. After the widget receives 'ssoJwt', it must be decoded by the widget. 
   5. The widget needs the value of 'ssoToken' in the payload of decoded 'ssoJwt' parameter to call the login service of  Yaghut.
   6. After calling the login service of the Yaghut, the widget receives 'sessionId'.
   7. With the 'sessionId' the widget can be call all banking service.
   
### ssoJwt
the 'ssoJwt' parameter is a JWT(JSON Web Tokens).
      
for example:
>     
      
       eyJhbGciOiJub25lIn0.eyJqdGkiOiJlNDBmY2VkYy0zNmYyLTQxOTYtOTc2Yi1mMWUxY2NhOWQwY2YiLCJiYW5rQ29kZSI6IkJPSU1JUiIsInNzb1Rva2VuRXhwaXJhdGlvblRpbWUiOjE1ODA3MTMxNTcyODUsImxhbmd1YWdlIjoiZmEiLCJzc29Ub2tlbiI6IjM5YmZjNDA2LWZiMDgtNDk1Zi05MTNjLTUyMWM4MTFlNTRhNCIsImV4cCI6MTU4MDcxMzQ1NX0.
       
this token must be encoded by the widget and its payload contains the below information:

```json
   {
      "jti": "e40fcedc-36f2-4196-976b-f1e1cca9d0cf",
      "bankCode": "BOIMIR",
      "ssoTokenExpirationTime": 1580713157285,
      "language": "fa",
      "ssoToken": "39bfc406-fb08-495f-913c-521c811e54a4",
      "exp": 1580713455
   }
```
> exp is the token expiration date. See [https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.4](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.4).

> jti is the JWT Id. See [https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7).

> bankCode is the swift code of bank.

> language is the selected language which is selected in Netway by the user.

> The widget can call  Yaghut login service with the ssoToken, clientId, clientSecret.(the service name in yahgut is: loginOauthClientSSOToken)

> ssoTokenExpirationTime is the ssoToken expiration date.

### Calling the Yaghut login service
To login in yaghut by soap protocol you can use wsdl like: 

> https://yaghut-api-url/yaghut/soap/soap_profileName?wsdl

for rest request you can call this URL:
> https://yaghut-api-url/yaghut/rest/tosan/user/loginOauthClientSSOToken 

> HTTP Method: POST

You must set (clientId, clientSecret, SSOToken) parameters in the body of request:

SOAP Sample:
```
soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://service.yaghut.modern.tosan.com/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:loginOauthClientSSOToken>
         <!--Optional:-->
         <context>
            <!--Zero or more repetitions:-->
            <data>
               <!--Optional:-->
               <key>?</key>
               <!--Optional:-->
               <value>?</value>
            </data>
         </context>
         <!--Optional:-->
         <request>
            <clientId>?</clientId>
            <clientSecret>?</clientSecret>
            <!--Optional:-->
            <loadCurrentValidSession>?</loadCurrentValidSession>
            <SSOToken>?</SSOToken>
         </request>
      </ser:loginOauthClientSSOToken>
   </soapenv:Body>
</soapenv:Envelope>
```

The body of response has a sessionId for specified user that login in netway and intract with current widget

```
/**
* شناسه جلسه کاری
*/
String sessionId;
Date lastLoginTime;//تاریخ و زمان آخرین ورود به سیستم
String name;//نام و نام خانوادگی
String foreignName;// مقدار این فیلد معادل نام است، نام نیز با توجه به زبان انتخابی مقدار خواهد گرفت
String title;//عنوان
Set<SubscriberUserInfoBean> subscribers;
String cif;//شماره مشتری
String mobile;//شماره موبایل
String email;//ایمیل
/**
* تاریخ انقضای سشن
*/
Date sessionExpirationDate;
	.
	.
	.
```
with this sessionId you can call some of banking services.

### Calling the deposit list service(a banking service)
To call the the deposit list service you must call this URL: 

for rest request you can call this URL:
> https://yaghut-api-url/yaghut/rest/tosan/deposit

> HTTP Method: POST


SOAP Sample:
```
soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://service.yaghut.modern.tosan.com/">
   <soapenv:Header/>
   <soapenv:Body>
      <ser:getDeposits>
         <!--Optional:-->
         <context>
            <!--Zero or more repetitions:-->
            <data>
               <key>SESSIONID</key>
               <value>806b5a9c-4e3e-4775-be07-bab0dac375a8</value>
            </data>
         </context>
         <!--Optional:-->
         <request>
            <length>10</length> 
            <offset>0</offset>
         </request>
      </ser:getDeposits>
   </soapenv:Body>
</soapenv:Envelope>
```

The body of response is a list of the below object

```
/**
* فهرستی از اطلاعات سپرده
*/
List<DepositBean> depositBeans;
/**
* تعداد کل سپرده های بازگشتی
*/
long totalRecord;
```

### Using Device Camera

#### Open Jetway application's camera module and scan qr code

First of all write a function named onMessage and assign this function to ReactNativeWebView object.
This function have a param filled by qr data and called when qr code scanned.

```
function onMessage(data, action) {
   document.getElementById("div").innerHTML = "Camera Data: " + data;
}
window.ReactNativeWebView.onMessage = onMessage;
```

Then create a js function to communicate with Jetway application for openning camera like this:

```
function openCamera() {
   var data = JSON.stringify({action: 'openCamera'});
   window.ReactNativeWebView.postMessage(data);
}
```

and call it like this:

```
<button onclick="openCamera()">Open Camera</button>
```

Full example:

```
<html lang="en">
   <head>
       <meta name="viewport" content="width=device-width, initial-scale=1">
       <script>
           function onMessage(data) {
               document.getElementById("div").innerHTML = "Camera Data: " + data;
           }
           window.ReactNativeWebView.onMessage = onMessage;
           function openCamera() {
               var data = JSON.stringify({action: 'openCamera'});
               window.ReactNativeWebView.postMessage(data);
           }
       </script>
   </head>
	<body>
		<h1 id="div">Camera: </h1>
		<button onclick="openCamera()">Open Camera</button>
	</body>
</html>
```

#### Open device camera using MediaDevices Web Api in Netway

For accessing device camera from a web page please read this link:

> https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia


### Using Digital Signing
##### Use digital signature to sign documents (PDF or TXT)

Write a function named onMessage and assign this function to ReactNativeWebView object. This function have a param filled by signed data and called when document signing is compelete.
```
function onMessage(data, action) {
               document.getElementById("div").innerHTML = "Document: " + data;
           }
           window.ReactNativeWebView.onMessage = onMessage;
           function signDocument() {
				//var dataToSign = convert PDF to base64 and pass as string / pass txt as string
				//var dataType = PDF or TXT
               var data = JSON.stringify({action: 'signDocument', dataToSign: 'dataToSign', dataType: 'pdf'});
               window.ReactNativeWebView.postMessage(data);
           }
```

and call this function like this:
```
<button onclick="signDocument()">Sign My Documetn</button>
```

Full example:
```
<html lang="en">
   <head>
       <meta name="viewport" content="width=device-width, initial-scale=1">
       <script>
           function onMessage(data) {
               document.getElementById("div").innerHTML = "Document: " + data;
           }
           window.ReactNativeWebView.onMessage = onMessage;
           function signDocument() {
				//var dataToSign = convert PDF to base64 and pass as string / pass txt as string
				//var dataType = PDF or TXT
               var data = JSON.stringify({action: 'signDocument', dataToSign: 'dataToSign', dataType: 'pdf'});
               window.ReactNativeWebView.postMessage(data);
           }
       </script>
   </head>
	<body>
		<h1 id="div">Document: </h1>
		<button onclick="signDocument()">Sign My Documetn</button>
	</body>
</html>
```

### Controll Phone Back Button
##### Controll when user pressed back button in Jetway application

Write a function named onMessage and assign this function to ReactNativeWebView object. This function has two param. The first param is for another Apis (like camera or digital signing) and second param is an action name.
If second param is equal to USER_BACK_PRESS, its means user pressed back button and you can do any action you want.
```
function onMessage(data, action) {
	if (action === 'USER_BACK_PRESS') {
		console.log('user pressed back button');
	}
}
window.ReactNativeWebView.onMessage = onMessage;
```

If you want to close WebView in app, first write a function like this:
```
function closeWebView() {
	var data = JSON.stringify({action: 'closeWebView'});
	window.ReactNativeWebView.postMessage(data);
}
```
And call it.


Full example:
```
<html lang="en">
   <head>
       <meta name="viewport" content="width=device-width, initial-scale=1">
       <script>
           function onMessage(data, action) {
               if (action === 'USER_BACK_PRESS') {
               		console.log('user pressed back button');
               }
           }
           window.ReactNativeWebView.onMessage = onMessage;

           function closeWebView() {
               var data = JSON.stringify({action: 'closeWebView'});
               window.ReactNativeWebView.postMessage(data);
           }
       </script>
   </head>
	<body>
		<button onclick="closeWebView()">Close WebView</button>
	</body>
</html>
```

### Download or share image (like receipts)
##### When you want to provide a button for download or share image files in phone in Jetway application.

First of all, you must provide base64 encoded string of your image and send it to application using given instructions.
For sharing image, write a function like this:

```
function shareImage() {
	var data = JSON.stringify({action: 'shareImage', base64: 'put_your_base64_here'});
	window.ReactNativeWebView.postMessage(data);
}
```

and for downloading image like this:

```
function shareImage() {
	var data = JSON.stringify({action: 'DOWNLOAD_IMAGE', base64: 'put_your_base64_here'});
	window.ReactNativeWebView.postMessage(data);
}
```

And call it.


Full example:
```
<html lang="en">
   <head>
       <meta name="viewport" content="width=device-width, initial-scale=1">
       <script>
           function shareImage() {
               var data = JSON.stringify({action: 'shareImage', base64: 'put_your_base64_here'});
               window.ReactNativeWebView.postMessage(data);
           }


           function downloadImage() {
               var data = JSON.stringify({action: 'DOWNLOAD_IMAGE', base64: 'put_your_base64_here'});
               window.ReactNativeWebView.postMessage(data);
           }
       </script>
   </head>
	<body>
		<button onclick="shareImage()">Share Image</button>
		<button onclick="downloadImage()">Download Image</button>
	</body>
</html>
```
   
