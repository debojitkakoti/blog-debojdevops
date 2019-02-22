---
layout: post
title:  "Difference between PEAR,PECL and Composer"
date:   2018-10-05 14:10:51 +0800
categories: Ops
tags: Ops
description: Difference between PEAR,PECL and Composer.
---

I was having hard time to understand difference between PEAR,PECL and composer and was not able to understand the proper use case or difference between these tools.So I am going to share some information regarding these. Correct me if it is wrong.

## PECL:( PHP Extension Community Library)

As the name suggests PECL is a repository for php extensions, You can download and install any php extension available from their repository.

## PEAR:( PHP Extension and Application Repository)

PEAR is a distribution system for reusable PHP libraries or classes.

## COMPOSER:

Composer is a dependency management tool in php.

Use case:

PEAR is soon going to be deprecated.

Using PECL you can install old php extensions, if you are not able to install those extensions from ubuntu repository. For example your server is running ubuntu LTS 14.04 and php version is 5.3, you will not be able to install oauth by apt-get install php5-oauth. It will give error.

You can use composer in your own project to manage external libraries and dependencies. Composer has https://packagist.org/ to use different php libraries.

##Steps to use pecl:
```
sudo apt-get install php-pear
sudo pecl channel-update pecl.php.net
sudo pecl install -f oauth-1.2.3
Here I am using oauth-1.2.3 because my php version is 5.3 . You can check the exact version of the extension need to install ,by searching the extension name at https://pecl.php.net/
```
Steps to use composer:

It is pretty much straight forward follow this [Composer](https://getcomposer.org/doc/00-intro.md)
