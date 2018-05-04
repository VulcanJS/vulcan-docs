---
title: Connecting Remotely
---

You might sometimes need to connect to your GraphQL endpoint (typically accessible at `http://yourapp.com/graphql`) from outside your app.

### Authorization Tokens

An easy way to perform remote API queries and mutations with special privilages is to emulate one of your app's users. You can do so by reusing their authorization token. 

To figure out the token, log in as the user you want to emulate and inspect any `graphql` request using your devtools' Network tab. In the Headers tab, locate the `Authorization` field of the Request Headers group and make a note of its value. 

![https://d3vv6lp55qjaqc.cloudfront.net/items/0i102x2t29322u0Y2H0L/%5B9c658a6cab34c2b07a48359f5003bc0f%5D_Screen%2520Shot%25202018-02-20%2520at%252017.58.06.png?X-CloudApp-Visitor-Id=f25ee64a8f9b32be400086060540ffac&v=221c3706](https://d3vv6lp55qjaqc.cloudfront.net/items/0i102x2t29322u0Y2H0L/%5B9c658a6cab34c2b07a48359f5003bc0f%5D_Screen%2520Shot%25202018-02-20%2520at%252017.58.06.png?X-CloudApp-Visitor-Id=f25ee64a8f9b32be400086060540ffac&v=221c3706)

This is the header and token you'll need to set when making your API call to the GraphQL endpoint. You can test it using a tool such as [GraphQL Playground](https://github.com/graphcool/graphql-playground)
.

In GraphQL Playground, select the "HTTP Headers" tab and type in (replacing the value with your own token): 

```
{ "Authorization": "JKdsfahyvfHvsdf234s59pNbHGH7Z2fadsdrfCmR8WimItX" }
```

### GraphQL Clients

You can send your GraphQL request as a plain text string (inspect the Request Payload object in your devtools' Network tab to see what that looks like) or you can use a GraphQL client to perform the request. 

Here are a few options: 

- Ruby: [graphql-client](https://github.com/github/graphql-client)
- Node: [graphql-request](https://github.com/graphcool/graphql-request)
- iOS: [apollo-ios](https://github.com/apollographql/apollo-ios) 
- Android: [apollo-android](https://github.com/apollographql/apollo-android)