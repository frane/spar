# Introduction

Spar, the *Single Page Application Rocketship*, is an opinionated framework that aims to ease single-page web-app development by addressing major challenges:

  * Asset organization
  * Compilation pipeline
  * Build & deployment support

Additionally, Spar provides templates & example projects for either a bare-bones project, or one with some of favorite tools pre-included.

Under the hood, Spar is a Rails-derived project, taking advantage of the powerful asset-pipeline, while stripping out legacy web-app support and introducing tools & patterns specific to single-page apps.

# Requirements

* Ruby 1.9.2-p125

# Installing Spar

```bash
$ gem install spar
````

If using `rbenv`:

```bash
$ rbenv rehash
```

# Getting Started

  Issue the following command to create a new Spar project:

```bash
$ spar myapp
$ cd myapp
$ spar server
```
  Your app will now be available at [http://localhost:8888](http://localhost:8888)

# Organization

  Spar apps are organized into the following folders and files:

    /app                      #Compiled assets that make your app
      /images
      /javascripts
        application.js.coffee #Your main JS output
      /stylesheets
        application.css.sass  #Your main CSS output
      /pages
        index.html.haml       #Your single, root HTML page
    
    config.yml                #ENV settings for dev, staging, prod
    Rakefile                  #Necessary for Heroku deploys
    README                    #your project's README
    
# Configuration

  `config.yml` defines your project's configuration for different environments. You may define any properties you like, which are available to you in your app directory files. 

  These settings may be overriden on a per-environment basis for `development`, `staging`, and `production` like so:

    default:    
      debug: true
      my_app_name: My App!
      my_api: http://localhost:8080

    staging:
      debug: false

    production:
      debug: false
      compress: true
      my_api: http://production-api.mysite.com

  Spar respects the following known configuration options:

  - `debug`: true/false, includes JS files individually for easier debugging
  - `digest`: true/false, adds MD5 hashes to end of filenames
  - `compress`: true/false, JS and CSS compression (uglify, and yahoo UI CSS compressor)

# Asset Pipeline

All asset files in the `app` directory are transformed through the Spar asset pipeline. Transformation first occurs for configuration properties defined in `config.yml`, followed by JS/CSS asset-compilation and composition.

## Configuration Property Replacement

First, configuration property substitution takes place according to your `config.yml` file. For instance, if your `index.html.haml` looks like this:

```haml
%html
  %head
    %title [{ my_app_name }]
```
first transforms to become:

```haml
%html
  %head
    %title My App!
```

## Compilation

After Spar performs configuration replacement, it then process files according to their extensions.

Inlcuded with Spar are transformations from:

- `file.html.haml` => `file.html`
- `file.js.coffee` => `file.js`
- `file.css.sass` => `file.css`
- `file.css.less` => `file.less`

## Dependency Management

Multiple Javascript (or CSS) files can be merged into a single file using the `require` and `require_tree` pre-processor directives.

If you want to serve one file, say `application.js`, that includes the content of multiple JS files, you may define an `application.js.coffee` like so:

```coffeescript
# This file will be compiled into application.js, which 
# will include all the files below required below

# The require directive can include individual file assets
# For instance, to include the compiled output of utils.js.coffee
#= require utils

# You can also recursively includes entire directories 
# using the require_tree directive
#= require_tree ./controllers
#= require_tree ./models
#= require_tree ./views
```
CSS files are composed similarly.

# Example Spar Applications

For your reference, and to build on top of, we've created two example applications using Spar.

The first is the quintessential TODO application as popularized by [addyosmani](http://addyosmani.github.com/todomvc/). The second, is a bootstrap application containing some of our favorite tools to kick-start the pretty (jQuery, Backbone, and Twitter Bootstrap)

Both can be found at our [spar-examples](https://github.com/BoundlessLearning/spar-examples) repo.

# Deploying Your Spar App

## Amazon Web Services

Spar has full support for S3 and CloudFront out of the box. First, add your AWS credentials to `config/production.rb`. You can look these up on the [AWS Security Credentials](https://portal.aws.amazon.com/gp/aws/securityCredentials) page.

```ruby
    set :acccess_key_id,          "my_access_key"
    set :secret_access_key,       "my+super+secret+access+key"
```

Next, you'll need visit the [AWS  S3 Console](https://console.aws.amazon.com/s3/home) and create a bucket to host your app. We suggest using the same as your fully qualified domain name. You should not use this bucket for anything else.

![click here](http://spar-screenshots.s3.amazonaws.com/s3_click_here.png)

![create bucket](http://spar-screenshots.s3.amazonaws.com/s3_create_bucket.png)

Specify your bucket in `config/production.rb`:

```ruby
    set :s3_bucket,               "app.example.com"
```

Next, you'll need to turn on [S3 Website Hosting](http://aws.typepad.com/aws/2011/02/host-your-static-website-on-amazon-s3.html) in the S3 console.

![bucket properties](http://spar-screenshots.s3.amazonaws.com/s3_bucket_properties.png)

![enable website](http://spar-screenshots.s3.amazonaws.com/s3_enable_website.png)

You'll need to create a new `CNAME` record. How this works is up to your hosting provider, but it should look something like this.

    app.example.com. IN CNAME app.example.com.s3-website-us-east-1.amazonaws.com.

### About Logging

You'll probably want to be able to log requests to your site. Even though your app uses cutting edge webscale tools like Airbrake, Loggly, Google Analytics, MixPanel, et al, eventually you'll want to know how many people hit you with IE6 or NoScript, and you gave them the middle finger.

Create a bucket log using the [AWS  S3 Console](https://console.aws.amazon.com/s3/home). Give it a name like `logs.example.com` and update either your app's CloudFront distribution or your app's S3 bucket to write its log files to this log bucket.

![enable logging](http://spar-screenshots.s3.amazonaws.com/s3_enable_logging.png)


### CloudFront

From here, it's easy to turn your fast site into a *really* fast site. From the [AWS  CloudFront Console](https://console.aws.amazon.com/cloudfront/home), create a new distribution *with the website form of your bucket name as the origin* and save the ID in `config/production.rb`.

```ruby
    set :cloudfront_distribution, "K9SYZF479EXMUAWH"
```
Take note of the **Domain Name** field (something like `d242ood0j0gl2v.cloudfront.net`). You will need to replace the CNAME you created earlier.

    app.example.com. IN CNAME d242ood0j0gl2v.cloudfront.net.

Now, every time you deploy, Spar will automatically issue CloudFront invalidation requests for index.html (and anything else without a hash value). CloudFront invalidations usually take around 8 minutes, but they can take quite a bit lot longer when Amazon is having problems.

## Apache, NginX, Lighttpd, etc

`rake assets:precompile` populates the `public/` directory with each view in its own `index.html`, which any web server should be able to serve with minimal configuration.

## GitHub Pages

    Coming Soon

## Heroku

    Coming Soon

# Issues & Bugs

Please use our Github [issue tracker](https://github.com/BoundlessLearning/spar/issues) for Spar.

# License

Spar is licensed under the terms of the MIT License, see the included LICENSE file.