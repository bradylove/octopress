---
layout: post
title: "Dead Simple Time Based OTP With Google Authenticator"
date: 2013-02-15 22:37
comments: true
categories: [Ruby, TOTP, Google, Authenticator, QRCode]
---
If you have ever been curious on how to add two factor authentication to your Rails/Sinatra app then today I will feed your curiosity. I am just going to show you the basics of creating a TOTP (Time based OTP) token, and creating a QRCode based on that token that can be used to add the token to your Google Authenticator on your favorite smart phone.

## Requirements
To be able to do this with the method I am showing here you will need to have Ruby, I tested this with Ruby 1.9.3-p385. You will also need to have libqrencode installed on your system. To do this visit [http://fukuchi.org/works/qrencode/index.html.en](http://fukuchi.org/works/qrencode/index.html.en) and download qrencode-3.4.1.tar.gz. I installed the library on Linux Mint 14 with the following instructions. This process may vary for your OS but these should work for most people.

    $ tar xzvf qrencode-3.4.1.tar.gz
    $ cd qrencode-3.4.1
    $ ./configure
    $ make
    $ sudo make install

Next you will need to grab a couple gems, these gems are awesome and make this process very painless. While we are at it we will also create `totp.rb` so we have somewhere to put our code.

    $ gem install rotp qrencoder
    $ touch totp.rb

## On with the code!
Then open `totp.rb` in your favourite text editor and enter in the following code. Once you have done that we will go through it and explain what each bit does.

``` ruby totp.rb
    require 'rotp'
    require 'qrencoder'

    random = ROTP::Base32.random_base32
    totp   = ROTP::TOTP.new(random)

    qr_string = totp.provisioning_uri("bradylove.com-love.brady@gmail.com")
    qr_code   = QREncoder.encode(qr_string)
    qr_code.png(pixels_per_module: 4, margin: 1).save("qr_code.png")

    loop do
      printf("What is your most excelent OTP? ")

      otp = gets.chop
      puts totp.verify_with_drift(otp, 5)
    end
```

## Break it down
Now that you have entered that in lets go through it. The first two lines require the `rotp` and `qrencoder` gems.

``` ruby
    require 'rotp'
    require 'qrencoder'
```

The next creates a 16 character base32 encoded random string. The next line creates our TOTP object which will handle the algorithms for creating and verifying the OTP.

``` ruby
    random = ROTP::Base32.random_base32
    totp   = ROTP::TOTP.new(random)
```

Next we use our TOTP object to create the URI with an `otpauth://` schema which ends up looking something like `otpauth://totp/bradylove.com-love.brady@gmail.com?secret=JBSWY3DPEHPK3PXP`, the `bradylove.com-love.brady@gmail.com` parameter we are passing it is just an identifier that will be displayed on the Google Authenticator as a label for the OTP.

``` ruby
    qr_string = totp.provisioning_uri("bradylove.com-love.brady@gmail.com")
```

Once we have that string we want to use it to create a QRCode. The QREncoder gem gives you plenty of options choices of ways to create the QRCode for this example we just go ahead and save it as a PNG image on the local directory.

``` ruby
    qr_code   = QREncoder.encode(qr_string)
    qr_code.png(pixels_per_module: 4, margin: 1).save("qr_code.png")
```

Finally to test the lines of code above we create an endless loop that will ask for the OTP from the user in the terminal and verify the given OTP. Where we verify the OTP we are calling `verify_with_drift` this will check the OTP based on Time.now +/- 5 seconds. This will make up for any timing issues between your computer and your phone.

``` ruby
    loop do
      printf("What is your most excelent OTP? ")

      otp = gets.chop
      puts totp.verify_with_drift(otp, 5)
    end
```

## Run it
So now that we have that all covered lets go ahead and test it. If you followed the code in this example, when we execute this script it will create a QRCode and save it to `qr_code.png` in the current directory. You can then open that image in your file browser, and scan it with your Google Authenticator, Google Authenticator will then display a new OTP every 30 seconds, check it out!

    $ ruby totp.rb
    What is your most excelent OTP? 707918
    true
    What is your most excelent OTP? 707918
    true
    What is your most excelent OTP? 707918
    false
    What is your most excelent OTP? 773428
    true

When you are done having OTP fun you can hit Ctrl-c to kill the process. This example is also available on my GitHub at [https://github.com/bradylove/blog-totp](https://github.com/bradylove/blog-totp). Now get back to work and smile!
