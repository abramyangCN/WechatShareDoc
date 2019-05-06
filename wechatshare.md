# WechatShare

## Requirement

1. **A verified wechat official account**
2. **AppID and AppSecret**
3. **Whitelist IP address & Domain Name.**

   Whitelist IP address:
   ![Alt Text](img/baseconfiguration.png 'baseconfiguration')
   ![Alt Text](img/whitelistip.png 'whitelistip')
   Whitelist Domain Name:
   Add the `.txt` file provided at the location shown above at root level for Tencent to confirm your ownership of the website.
   ![Alt Text](img/functionsetting.png 'function setting')
   Download the `.txt` file and add Domain name in the blanks
   ![Alt Text](img/adddomain.png 'adddomain')
   **Note**: The Domain name list can be edit three times per month.

   This configuration send the http `get` request to wechat server so that the wechat server can return the signature. [click here for more detail](#backend)

## Steps

---

1. Get the AppID and AppSecret in settings from wechat platform dashboard.
2. Get access_token through wechat API with AppID and AppSecret,and Verify the configuration via the config interface.[(Generated at Backend)](#backend)
3. Add the Api to api list which will be used.(#frontend)

## Backend

---

###Get `ACCESS_TOKEN`

> https method: GET
> https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET

Wechat will return a json:

`{"access_token":"ACCESS_TOKEN","expires_in":7200}`

1. The current effective period for the `ACCESS_TOKEN` will be communicated from the returned expire_in, which at present is a value within 7,200 seconds.

2. Then use `ACCESS_TOKEN` of step 1 to get the `ticket`,

   > https method: GET
   > https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi

   And connect `ticket`, `nonceStr`, `timestamp` and `url` to obtain `signature` [click here for more detail](#signature-algorithm) .
   [\>>>Wechat wiki](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319)

The server should recieve the data:

```json
{
  "errcode": 0,
  "errmsg": "ok",
  "ticket": "bxLdikRXVbTPdHSM05e5u5sUoXNKd8-41ZO3MhKoyN5OfkWITDGgnr2fwJ0m9E8NYzWKVZvdVtaUgWvsdshFKA",
  "expires_in": 7200
}
```

###Signature algorithm

The rules of generate signature is as follow: the fields of involved signature included noncestr(random string), valid jsapi_ticket, timestamp(time stamp), url (current web page URL, not including # and the remaining ). After all the parameters are signed, sort ASCII code according to the field name from small to large (dictionary order). Use the format of URL key-value pairs (key1 = value1&key2=value2 ...) connect into string1. It should be noted here that all parameter names are lowercase characters. String1 as sha1 encryption, field names and field values are the original value, without URL transfer.

Therefore, signature=sha1(string1). Example:

```javascript
noncestr=Wm3WZYTPz0wzccnW //ramdom string
jsapi_ticket=sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg //from wechat server
timestamp=1414587457 //timestamp
url=http://mp.weixin.qq.com?params=value //from front-end, not static,to make sure that the url of backend and frontend must be the same.
```

Step 1. After all the parameters are signed, sort ASCII code according to the field name from small to large (dictionary order). Use the format of URL key-value pairs (key1=value1&key2=value2 ...) connect into string1.

```javascript
jsapi_ticket=sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg&noncestr=Wm3WZYTPz0wzccnW&timestamp=1414587457&url=http://mp.weixin.qq.com?params=value
```

Step 2. Use string1 to sign sha1, to obtain signature：

`0f9de62fce790f9a083d5c99e95740ceb90c27ed`

![Alt Text](img/processon.png 'processon')

Note the following points:

1. **Limitation**: Getting Access_Token is limited 2000 times per day.
2. The noncestr and timestamp used for signature must be **the same** as nonceStr and timestamp in wx.config.
3. The urlused for signature must be the complete page URL to call JS interface
4. For security reasons, developers must implement the logic of signature on the server.
5. If invalid signature or other errors occurred, see appendix 5 for Common errors and solutions

##Frontend

---

1. **Import JS file  
   In need to use JS interface page, import JS file as below (support https): http://res.wx.qq.com/open/js/jweixin-1.2.0.js**
2. **Verify the configuration via the config interface
   All pages need to use JS-SDK must first import configuration information.**
   **For example:**

   ```javascript
   wx.config({
     debug: true,
     appId: 'data.appid', // Required, the only identification of Official account.
     timestamp: 'data.timestamp', // Required, generate a signed timestamp
     nonceStr: 'data.nonceStr', // Required, generate a signed nonceStr
     signature: 'data.signature', // Required, signature. See Appendix 1
     jsApiList: ['onMenuShareTimeline', 'onMenuShareAppMessage'] // Required, required JA interface list, all JS interface list, see Appendix 2
   });
   wx.ready(function() {
     // config information validation will execute by the ready method, all interface call must be done after config interface get the result. Config is an asynchronous operation of client's side. If call relevant interface is required when loading a page, the relevant interface must be called on the ready function to ensure correct execution. For those interface call when triggered by users, it can be called directly, not required to be in the ready function.
   });
   wx.error(function(res) {
     // config information failure will execute error function, such as expired signature to cause verification failure, the error message can be viewed via debug mode in config, also be viewed via returned res parameter. SPA can renew signature here.
   });
   ```

3. **Share to moment**

   ```javascript
   wx.onMenuShareTimeline({

       title: '', // Share title
       link: '', // Share link, this link domain name and path must be the same as the current page which corresponding to JS secured domain name as Official account
       imgUrl: “,// Share icons.
       success: function () {
           // The user confirms the callback function that was executed after sharing
       },
       cancel: function () {
           // The user cancels the callback function that was executed after sharing
       }
   });
   ```

4. **Share to Friends**
   ```javascript
   wx.onMenuShareAppMessage({
     title: '', // Share title
     desc: '', // Share description
     link: '', // Share Link,this link domain name and path must be the same as the current page which corresponding to JS secured domain name as Official account
     imgUrl: '', // Share Icon
     type: '', // Share type, music, video link, not filled default link
     dataUrl: '', // If type is music or video, provide data links, the default is empty
     success: function() {
       // The user confirms the callback function that was executed after sharing
     },
     cancel: function() {
       // The user cancels the callback function that was executed after sharing
     }
   });
   ```

```

```
