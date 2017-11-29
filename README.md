# OmniAuth::Amazon
[![Build Status](https://travis-ci.org/wingrunr21/omniauth-amazon.png)](https://travis-ci.org/wingrunr21/omniauth-amazon) [![Gem Version](https://badge.fury.io/rb/omniauth-amazon.png)](http://badge.fury.io/rb/omniauth-amazon)

[Login with Amazon](https://login.amazon.com/) OAuth2 strategy for OmniAuth 1.0

## Installation

Add this line to your application's Gemfile:

    gem 'omniauth-amazon'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install omniauth-amazon

## Prereqs

You must create an application via the [Amazon Developer Console](https://sellercentral.amazon.com). Once that is complete, register two URLs under <i>Web Settings ->  Allowed JavaScript Origins and Allowed Return URLs</i>:

Allowed JavaScript Origins:
```
    http://localhost:3000
    https://your_website_here
```

Allowed Return URLs:
```
    http://localhost:3000/users/auth/amazon/callback
    https://your_website_here/users/auth/amazon/callback
```

Amazon requires HTTPS for the whitelisted callback URL (except localhost). They don't appear to
like ```.dev``` domains too much but happily accept localhost.

## Usage
Usage is similar to other OAuth2 based OmniAuth strategies:
```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
    provider :amazon, ENV['AMAZON_CLIENT_ID'], ENV['AMAZON_CLIENT_SECRET'],
        {
          :scope => 'profile postal_code' #default scope
        }
end
```
## Example
1. Add Amazon client, secret, and callback to omniauth config `config/initializers/devise.rb`.

I used Heroku, but you can use anything for production, just point the callback URL correctly back to your server.
Usage is similar to other OAuth2 based OmniAuth strategies with devise:

```ruby
  if Rails.env.production?
    config.omniauth :amazon, ENV['AMAZON_CLIENT_ID'], ENV['AMAZON_CLIENT_SECRET'], 
                    callback_url: "https://your_website_here/users/auth/amazon/callback"
  else
    config.omniauth :amazon, ENV['AMAZON_CLIENT_ID'], ENV['AMAZON_CLIENT_SECRET'], 
                    callback_url: "http://localhost:3000/users/auth/amazon/callback"
  end
```
2. Add `amazon` as a provider to the devise `attr` in `User` model, for example `:omniauthable, :omniauth_providers => [:amazon]`:
```ruby
class User < ActiveRecord::Base
  ...
  devise :database_authenticatable, :registerable, 
         :recoverable, :rememberable, :trackable, :validatable,
         :omniauthable, :omniauth_providers => [:amazon]
  ...
end
```

3. Create omniauth controller in `users` controller directory (it is the only file I have in there) `app/controllers/users/omniauth_callbacks_controller.rb`:

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def amazon
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Amazon") if is_navigational_format?
    else
      session["devise.amazon_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to unauthenticated_root_path
  end
end
```

4. Add method to `User` model to add record after successfully returning to callback URL from omniauth:

```ruby
Class User < ApplicationRecord
  devise :database_authenticatable, :registerable, 
         :recoverable, :rememberable, :trackable, :validatable,
         :omniauthable, :omniauth_providers => [:amazon]
         
  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
      user.provider = auth.provider
      user.name = "#{auth.info.first_name} #{auth.info.last_name}"
      user.uid = auth.uid
      user.email = auth.info.email
      user.password = Devise.friendly_token[0,20]
    end
  end
  ...
  
  def password_required?
    super && self.provider.blank?
  end

  def update_with_password(params, *options)
    if encrypted_password.blank?
      update_attributes(params, *options)
    else
     super
    end
  end

  def has_no_password?
    self.encrypted_password.blank?
  end
end
```

5. Add a button to the login page, with devise you add it to the `views/devise/shared/_links.html.erb` file
```ruby
<%- resource_class.omniauth_providers.each do |provider| %>
  <%= link_to "Sign in with #{OmniAuth::Utils.camelize(provider)}", omniauth_authorize_path(resource_name, provider) %>
<% end -%>
```
## Configuration

Config options can be passed to `provider` via a `Hash` and can be added to the `config.omniauth` from Step 1. of the Example section:

* `scope`: A space-separated list of permissions. Can be `profile`,
  `postal_code`, `profile:user_id`, or a combination of options.  
  Defaults to: `profile postal_code`
    * Requesting the `profile:user_id` scope will not display an additional consent
      screen the first time the user logs in.


## Resources
* [Login with Amazon button guide](https://login.amazon.com/button-guide)
* [Login with Amazon style guide](https://login.amazon.com/style-guide)

## Todo
1. Fix ```raw_info``` to see why ```client.request``` has to be used in query
   mode

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
