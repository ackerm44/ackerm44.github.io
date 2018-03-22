---
layout: post
title:      "User Auth with JWT, Starting with Rails API"
date:       2018-03-20 17:50:10 -0400
permalink:  react_redux_with_rails_api
---


Building a React app from the ground up has really began to solidify my undestanding of the React library.  The challenging part of the application has been to instantiate user authentication.  I decided to implement an application that displays SpaceX data and wanted to take the app to the next level by practicing with user authentication using JWT (JSON Web Token).  

Whereas we generally use sessions and cookies to determine on each page whether a user is logged in, on a single page application, we are not physically moving to different pages but rather rendering different content on the same page.  This makes using sessions difficult.  

JSON Web Token is a great solution for single page applications that need to have a layer of authentication.  A JWT is comprised of three parts:
1. A Header: the header includes the algorithm used to encode and decode the "signature" as well as the type of token which in this case is "jwt".
2. A Payload: The payload includes the information you want to sign.  If, for example, JWT is being used for logging, this information would include the username and password perhaps.
3. A Signature: The signature is put together with the encoded Header and the encoded Payload with a secret key.  It is encoded in this way to verify that the message was not changed along the way.  

Utilizing JWT for the first time was not the most intuitive task I've ever performed but now that I've implemented it, I've found it much easier to see how JWT functions.  I followed these two wonderful blog posts fairly closely in order to implement the tokens on both the server side and the client side: 

* http://www.thegreatcodeadventure.com/jwt-auth-in-rails-from-scratch/
* http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/

These posts are slightly dated by now so note that there have been some changes in both Rails and React. I also implemented my own changes to follow the logic flow better in my head.  Its possible my implementation will have some issues in production, but for development purposes at this point, it appears to be operating correctly.  

Lets gets started.  

It is easiest to start on the server side.  Lets discuss the steps needed to implement JWT on the Rails side:

*  Add  gem 'jwt' to your Gemfile.  There are a few methods packaged with this gem that we will use to encode and decode our JWT. You will also want a way to hide a secret key that we are about to generate.  I chose to use gem 'dotenv-rails'. If you haven't used before, check out the documentation on implementing and **be sure to add your .env file to your .gitignore file.**   Lastly, you will need 'bcrypt' installed if you haven't already.  Bcrypt will need a user model with users table that has atleast a password and password_confirmation column defined and the macro method has_secure_password in the user model.  Check out documentation for bcrypt as well if unfamiliar.  
*  Bundle install.  
*   Create a new file in your lib directory called 'auth.rb'.  I also in config/application.rb added config.autoload_paths << Rails.root.join('lib').
* We need to create a hashing key that will be used essentially like a password to decode and encode our token.  There are several way to generate this key but a simple way is to do so in pry.  Enter into pry and type: require 'digest'.  Then enter the following with your own digestable word: Digest::SHA256.digest('spring').  'spring' can be anything.  This will return a string with our secret like so:

```
// ♥ pry
[1] pry(main)> require 'digest'
=> true
[2] pry(main)> Digest::SHA256.digest('spring')  
=> "b*IM>\xA8\xC7\xBA/\xEDO7\x90\x9F\x14\xD9\xB5\n\xB4\x122-\xE3\x9B\xE6,\x8Dl$\x18\xBF\xCA"
```

*   Back to our auth.rb file, we will want to create two methods that issue our encoded authorization and then a method that will decode to get access to the user id:

```
# lib/auth.rb  
require 'jwt'

class Auth
  ALGORITHM = 'HS256'
	
  def self.issue(payload)
    JWT.encode(
      payload,
      auth_secret,
      ALGORITHM)
  end
	
  def self.decode(token)
    JWT.decode(token, 
      auth_secret, 
      true, 
      { algorithm: ALGORITHM }).first
  end
	
  def self.auth_secret
    ENV["AUTH_SECRET"]
  end
	
end 
```

You can see that outside of these two methods, we have defined the algorithm that is used to decode our token as well as the secret key.  The secret key is wrapped in ENV since in my case, I am accessing this variable in my .env file which is hidden away and unaccessible.  

* Next we need to create an api route so that our client side will have access to the database of users to either create a new user or check that a user exists.  In config/routes: add ` post '/login', to: "sessions#create"`.  In my application, I am accessing all database routes through an /api/ prefix so I've added it like so: 

```
scope '/api' do
     post :login, to: 'sessions#create'
end
```

This means that the client side will access the user table by going to localhost:3000/api/login. 

* Next with this route, we need to build the session controller with the create action. 

```
def create
    user = User.find_by(username: auth_params[:username])
    if user && user.authenticate(auth_params[:password])
      jwt = Auth.issue({user: user.id})
      render json: {jwt: jwt}
    else
      render json: {:errors=>
        [{:detail=>"incorrect email or password",
          :source=>{:pointer=>"user/err_type"}}
        ]}, status: 404
    end
  end

  private
    def auth_params
      params.require(:auth).permit(:username, :password)
    end
```

Here I am finding the user by taking in the username from the auth_params defined.  If you are not asking for a username, you may have another identifier here. If the user exists, I then verify the password.  If all is good, we issue a token from our Auth file we defined earlier and render json with this jwt variable.  If there's a problem, an error will be rendered instead.  

*  Next we need to create a route, controller and action for when a new user is signing up.  Again, define a route in config/routes like so:     `post :signup, to: 'users#create'`  (scoped under "/api") and in your users_controller, define the create action: 

```
  def create
    user = User.new(user_params)
    if user.save
      jwt = Auth.issue({user: user.id})
      render json: {jwt: jwt}
    end
  end

  private
  def user_params
    params.require(:auth).permit(:username, :email, :password, :password_confirmation)
  end
```

This is performing similar actions as the sessions#create action but is not checking for validity of the password since this is a new user.  In the end, we are still returning the json of the jwt variable issued.  

*  Finally, we need to perform some checks on the authenticity of a user while they are logged in to determine if they are authenticated and able to visit certain or all parts of our application.  The idea is that while a user is traversing a site, when they visit a particular route, their request for that route will include an authorization header key with the encoded jwt information which looks something like this:

```
Request Headers

Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9,la;q=0.8
authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxfQ.gp5MJJh3FHzeQ2cU695I.......
Connection: keep-alive
Content-Length: 100
content-type: application/json
Host: localhost:3000
Origin: http://localhost:3000
Referer: http://localhost:3000/past/22
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.162 Safari/537.36
```
(To see your request headers, visit the network tab in in DevTools in the browser, click on headers and scroll down)

We will define how to set this authorization header in my next post, but for now, know that this will exist in requests that require authorization.  

Therefore, in the application controller, we need to define checks for these headers:

```
class ApplicationController < ActionController::API

  def logged_in?
    !!current_user
  end

  def current_user
    if auth_present?
      user = User.find(auth["user"])
      if user
        @current_user ||= user
      end
    end
  end

  private
  def token
    request.headers['authorization'].scan(/Bearer (.*)$/).flatten.last
  end

  def auth
    Auth.decode(token)
  end

  def auth_present?
    if request.headers['authorization'] != "Bearer undefined"
      !!request.headers['authorization']
    else
      false
    end
  end
end
```

I differ a bit from the tutorial in this example.  After hours trying to determine why auth_present? was returning nil, I decided to take a more direct approach and check directly for a key in request.headers called 'authorization'.  If a user is not signed in, my application will return "Beared undefined".  As long as that doesn't exist, that means a user is signed in and an auth header that looks like 'Bear easlkjlsjf....' will be defined instead, letting auth_present? return a true value.  In the current_user method, now that auth_present? is true, we then find the user id by decoding the token from our authorization request header using our Auth.decode method defined in lib/auth.rb. The token gets stripped of its 'Bearer' tag and just returns the encoded token.  This then returns to us something like {user: 1}.  


In my next post, I'll discuss hooking up our client side.  


