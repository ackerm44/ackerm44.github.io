---
layout: post
title:      "React + Redux with Rails API"
date:       2018-03-20 21:50:09 +0000
permalink:  react_redux_with_rails_api
---


Building a React app from the ground up has really began to solidify my undestanding of this library from what before was rather shaky but the challening part of the application has been to instantiate user authentication.  Although I decided to implement an application that displays SpaceX data, I wanted to take the app to the next level and practice with user authentication using JWT (JSON Web Token).  

JSON Web Token is a great solution for single page applications that need to have a layer of authentication.  A JWT is comprised of three parts:
1. Header: the header includes the algorithm used to encode and decode the "signature" as well as the type of token which in this case is "jwt"
2. Payload: The claims includes the information you want to sign.  If JWT is being used for signing in for example, this information would include the username and password perhaps
3. Signature: The signature is put together with the encoded Header and the encoded Payload with the secret.  It is encoded in this way to verify that the message was not changed along the way.  



