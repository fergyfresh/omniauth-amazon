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

## Configuration

Config options can be passed to `provider` via a `Hash`:

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


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/wingrunr21/omniauth-amazon/trend.png)](https://bitdeli.com/free "Bitdeli Badge")
