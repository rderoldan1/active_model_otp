[![Build Status](https://travis-ci.org/heapsource/active_model_otp.png)](https://travis-ci.org/heapsource/active_model_otp)

# ActiveModel::Otp

**ActiveModel::Otp** makes adding **Two Factor Authentication**(TFA) to a model simple, let's see what's required to get AMo::Otp working in our Application, using Rails 4.0 (AMo::Otp is also compatible with Rails 3.x versions) we're going to use an User model and some authentication to it. Inspired in AM::SecurePassword

## Installation

Add this line to your application's Gemfile:

    gem 'active_model_otp'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install active_model_otp

## Setting your Model

We're going to add a field to our ``User`` Model, so each user can have an otp secret key. The next step is to run the migration generator in order to add the secret key field.

```ruby
rails g migration AddOtpSecretKeyToUsers otp_secret_key:string
=>
      invoke  active_record
      create    db/migrate/20130707010931_add_otp_secret_key_to_users.rb
```

We’ll then need to run rake db:migrate to update the users table in the database. The next step is to update the model code. We need to use has_one_time_password to tell it will be use TFA.

```ruby
class User < ActiveRecord::Base
  has_one_time_password
end
```


##Usage

The has_one_time_password sentence provides to the model some useful methods in order to implement our TFA system. AMo:Otp generates one time passwords according to [RFC 4226](http://tools.ietf.org/html/rfc4226) and the [HOTP RFC](http://tools.ietf.org/html/draft-mraihi-totp-timebased-00). This is compatible with Google Authenticator apps available for Android and iPhone, and now in use on GMail.

The otp_secret_key is saved automatically when a object is created, 

```ruby
user = User.create(email: "hello@heapsource.com")
user.otp_secret_key
 => "jt3gdd2qm6su5iqh" 
```

**Note:** You can fork the applications for [iPhone](https://github.com/heapsource/google-authenticator) & [Android](https://github.com/heapsource/google-authenticator.android) and customize it

### Getting current code (ex. to send via SMS)
```ruby
user.otp_code # => '186522'
sleep 30
user.otp_code # => '850738'
```

### Authenticating using a code

```ruby
user.authenticate_otp('186522') # => true
sleep 30 # let's wait 30 secs
user.authenticate_otp('186522') # => false
```

### Authenticating using a slightly old code

```ruby
user.authenticate_otp('186522') # => true
sleep 30 # lets wait again
user.authenticate_otp('186522', drift: 60) # => true
```

## Google Authenticator Compatible

The library works with the Google Authenticator iPhone and Android app, and also includes the ability to generate provisioning URI's to use with the QR Code scanner built into the app.

```ruby
# Use you user's emails for generate the provisioning_url
user.provisioning_uri # => 'otpauth://totp/hello@heapsource.com?secret=2z6hxkdwi3uvrnpn'

# Use a custom fied for generate the provisioning_url
user.provisioning_uri("hello") # => 'otpauth://totp/hello?secret=2z6hxkdwi3uvrnpn'
```

This can then be rendered as a QR Code which can then be scanned and added to the users list of OTP credentials.

### Working example

Scan the following barcode with your phone, using Google Authenticator

![QRCODE](http://qrfree.kaywa.com/?l=1&s=8&d=otpauth%3A%2F%2Ftotp%2Froberto%40heapsource.com%3Fsecret%3D2z6hxkdwi3uvrnpn)

Now run the following and compare the output

```ruby
require "active_model_otp"
class User
  extend ActiveModel::Callbacks
  include ActiveModel::Validations
  include ActiveModel::OneTimePassword

  define_model_callbacks :create
  attr_accessor :otp_secret_key, :email

  has_one_time_password
end
user = User.new
user.email = 'roberto@heapsource.com'
user.otp_secret_key = "2z6hxkdwi3uvrnpn"
puts "Current code #{user.otp_code}"
```

**Note:** otp_secret_key must be generated using RFC 3548 base32 key strings (for compatilibity with google authenticator)

### Useful Examples

#### Generating QR Code with Google Charts API

#### Generating QR Code with rqrcode and chunky_png

#### Sendind code via email with Twilio

#### Using with Mongoid

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
