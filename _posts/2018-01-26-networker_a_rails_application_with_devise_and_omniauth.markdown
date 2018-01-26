---
layout: post
title:      "Networker: A Rails Application with Devise and OmniAuth"
date:       2018-01-26 21:24:37 +0000
permalink:  networker_a_rails_application_with_devise_and_omniauth
---


Creating a rails application from scratch proved to be a heftier project than I first imagined though what feels like a solid application in the end.

After two days of work and not feeling excited about my original app idea, I decided to pause and draw out any thoughts on other domains that could be more useful.  Thinking on what will be coming down my pipeline in the next few months, I decided to create a networking event tracker that allows a user to tracks networking events and detail if follow ups are needed for that event.  It also maintains organizations associated with the event and contacts.  

Since I was feeling most apprehensive about Devise and OmniAuth, I decided to commit myself to integrating these into my application so I could learn them better.  They both proved to not be as scary as I originally thought.  

**Devise **

Devise has very good documentation for customizing views and functionality so applying it just took some patience and a lot of reading.  One lesson on Devise I learned early on is how to integrate another field on the sign-up page. Devise defaults to just asking for email and password but I wanted the user's name to also be included in the form.  To add another field, you would do so like normal on the views/devise/registrations/new.html.erb view.  In order to pass the name data through to the database however, you do need to whitelist it, but in a slightly different way than normal.  In the ApplicationController, add:

`before_action :configure_permitted_parameters, if: :devise_controller?`

then define it with:

```
def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
end
```

Devise will now recognize that additional field and save it to the database.



**OmniAuth with Github**

In regards to OmniAuth with Devise, this was a bit more trial and error.  

To start, install `gem 'omniauth-github`'  

I also installed the `gem 'dotenv-rails'` to provide security for the github client id and client secret.  

Run bundle install.

Next we need to add a few columns to our users table.  Run `rails g migration AddOmniauthToUsers provider:string uid:string` and then run `rake db:migrate`.

Next visit the config/initializers/devise.rb file and add this line to it:

`config.omniauth :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET'], scope: 'user,public_repo'`

In your User model, you will also need to make it omniauthable by adding: 

`devise :omniauthable, omniauth_providers: %i[github]`

Run rake routes and you will see that two new routes have been added, one for redirecting to github for authorization and one for the callback to return to your site.  

In your New Registration view, add a link for the user to click on:

```
<%= link_to "Sign in with Github", user_github_omniauth_authorize_path %>
```

The Gem 'dotenv-rails' allows us to set variables that can be accessed in the .env file and then passed to the devise initializer.  Create a file called .env in the root of the project directory and add two lines: 

```
export GITHUB_KEY = *yourclientid*
export GITHUB_SECRET = *yourclientsecret*
```

It is also recommend tht you add .env to your .gitignore file. 

Next visit github to get your actual clientid and client secret keys.  Go to settings through your github account, click on Developer Settings and then click Register OAuth App or Register New App.  Here you will enter your app's name, the homepage (in development it is just http://localhost:3000), a quick description and finally, the callback url.  When using Devise, the callback will look something like this:  http://localhost:3000/users/auth/github/callback.  You can know for sure what your route is by running rake routes.  Look for the callback_path.  

Save your application.  There are a few final tasks to get this working which is to tell your app what to do when github replies with a valid user and redirects back to your site.  In config/routes.rb, update the users route to looke like this:

`devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }`

We then need to add the controller which is namespaced through Users.  Create a users file under controllers if one does not exist, and add omniauth_callbacks_controller.rb file.  In the file, add:

```
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def github
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, event: :authentication 
      set_flash_message(:notice, :success, kind: "Github") if is_navigational_format?
    else
      session["devise.github_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to new_user_registration_url
  end
end
```

The method from_omniauth then needs to be defined in your User model with something like this:

```
def self.from_omniauth(access_token)
    data = access_token.info
    user = User.where(email: data['email']).first
    unless user
        user = User.create(name: data['name'],
           email: data['email'],
           password: Devise.friendly_token[0,20]
        )
    end
    user
  end
```


And that should do it.  There is of course so much more that can be customized with both Devise and OmniAuth but this will get you started.  
<hr>
To see and use this application, visit https://github.com/ackerm44/networker


