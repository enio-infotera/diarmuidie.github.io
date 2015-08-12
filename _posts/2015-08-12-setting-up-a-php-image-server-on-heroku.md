---
layout: post
title: 'On The Fly PHP Image Manipulation Server With ImageRack, Heroku and S3'
tags:
    - Guide
excerpt: "This guide will show you how to install an ImageRack server on an free Heroku dynamo to resizing and serve images from an S3 bucket."
---
![Resized image from a live ImageRack Server on Heroku](http://imagerack.herokuapp.com/large/photo1.jpg)

This guide will show you how to install an [ImageRack](https://github.com/diarmuidie/ImageRack) server on an [Heroku](https://www.heroku.com/) free dynamo to resize and serve images from an [Amazon S3](https://aws.amazon.com/s3/) bucket. ImageRack is an easy to setup on the fly image manipulation server written that I wrote to simplify handling user media uploads.

The companion code for this guide is available on BitBucket: [bitbucket.org/diarmuidie/imagerack-heroku](https://bitbucket.org/diarmuidie/imagerack-heroku/).

Before we begin there are a few prerequisites:

- An [Heroku](https://www.heroku.com/) account.
- [Heroku toolbelt](https://toolbelt.heroku.com/) installed and working.
- An [AWS S3](https://aws.amazon.com/s3/) bucket and its corresponding **key** and **secret**.
- PHP and [Composer](https://getcomposer.org/) installed on your local/dev machine.

Step 1 - Setting up ImageRack
-----------------------------

To start let's get a copy of the ImageRack server on our local machine:

```bash
$ composer create-project diarmuidie/imagerack heroku-imagerack
```

This will download and install the ImageRack server and its dependencies in the `heroku-imagerack` folder. We also need to install the [Flysystem S3 adapter](http://flysystem.thephpleague.com/adapter/aws-s3-v3/) because we will be storing our source images there:

```bash
$ composer require league/flysystem-aws-s3-v3
```

Now that everything is downloaded we can begin to configure the server.

Copy the `bootstrap/dependencies.s3.sample.php` file and save it as `bootstrap/dependencies.php`. This file is setup to use S3 as the source and cache locations but we only want to use S3 as the source. Cached images will be stored locally on the Heroku dynamo.

You will have to edit the cache dependency line to change it from:

```php
$dependencies['cache'] = new Filesystem(new AwsS3Adapter($s3Client, 'test-image-rack-cache'));
```

To:

```php
$dependencies['cache'] = new Filesystem(
    new League\Flysystem\Adapter\Local(__DIR__.'/../storage/cache')
);
```

And that's it! There are more config option in ImageRack that you can tweak but the defaults will work just fine.


Step 2 - Preparing ImageRack to Run on Heroku
---------------------------------------------

There are a few steps to getting our PHP to run on Heroku.

First we need to update the S3 connection settings (in `bootstrap/dependencies.php`) to use the Heroku [config vars](https://devcenter.heroku.com/articles/getting-started-with-php#define-config-vars) (we will set these later):

Edit the S3Client Factory credentials to use `getenv()` like below:

```php
$s3Client = S3Client::factory([
    'credentials' => [
        'key'    => getenv('S3_KEY'),
        'secret' => getenv('S3_SECRET'),
    ],
    'region' => 'us-east-1',
    'version' => 'latest',
]);
```
Don't forget the edit the `region` option to correspond to the region that your bucket is in.

We also need to create a `Procfile` to tell Heroku what kind of server this is and where the document root folder is. Save the file to the project root as `Procfile` with the following content:

```
web: vendor/bin/heroku-php-apache2 public/
```

Finally we can edit the `composer.json` `require` section to make sure Heroku installs the correct versions of PHP and GD:

```JSON
"require": {
    "php": ">=5.4",
    "ext-gd" : "*",
    ...
}
```
After editing the `composer.json` file you need to run an update to keep the lock file in sync:

```bash
$ composer update
```

Step 3 - Deploying to Heroku
-----------------------------

To be able to deploy to Heroku our files must be in a git repo. Initialise a git repo and add all the files to it:

```bash
$ git init
$ git add .
$ git commit -m 'Initial commit'
```
Now we can finally create a Heroku app:

```bash
$ heroku create
```
If that all went OK Heroku will output a URL for your new project (something like: http://sharp-rain-871.herokuapp.com/).

In order for ImageRack to connect to S3 you need to setup the config vars in Heroku:

```bash
$ heroku config:set S3_KEY=<put your S3 key here>, S3_SECRET=<put your S3 secret here>
```
Now we can push the whole project to Heroku:

```bash
$ git push heroku master
```
Once that completes you should be able to access the Heroku ImageRack server using:

```bash
$ heroku open
```

The first time the app loads it will take a few seconds as the Dynamo wakes up. Once you have uploaded some images to your S3 bucket you can access them at `http://<heroku-app-name>.herokuapp.com/<template>/path/to/image.jpg` . Where `template` is a template [you create](https://github.com/diarmuidie/ImageRack#templates) or the default included template (`small`) and `path/to/image.jpg` is the path to the image file on your S3 bucket.

If you get stuck along the way have a look at the sample repo I created [bitbucket.org/diarmuidie/imagerack-heroku](https://bitbucket.org/diarmuidie/imagerack-heroku/) and the Heroku help docs [devcenter.heroku.com/articles/getting-started-with-php](https://devcenter.heroku.com/articles/getting-started-with-php).
