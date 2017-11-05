---
layout: post
title: 'Downloading Sony GPS Ephemeris Data on macOs'
tags:
    - Article
excerpt: "CLI tool to download GPS ephemeris data to Sony Cyber-shot cameras on MacOS"
---

## What is GPS Ephemeris Data

GPS ephemeris data is used by your GPS devices (smartphone, GPS running watch, camera with built in GPS tagging etc.) to predict which satellites will be in the sky at a particular point in time. Knowing which satellites are available dramatically decreases the amount of time it takes for your device to get a GPS "fix". The prediction data is usually valid only for a few days so new data needs to be downloaded from the internet periodically.

## What is the issue with Sony Cameras on macOS
Normally the GPS ephemeris data file is downloaded by the [Sony PlayMemories software](http://support.d-imaging.sony.co.jp/www/disoft/int/download/playmemories-home/mac/en/) application and loaded onto your camera each time you connect it to your computer, however it [no longer works with macOS 10.13](http://sony-eur-eu-en-web--eur.custhelp.com/app/answers/detail/a_id/143062/~/macos-10.13-%28high-sierra%29-compatibility-information-for-application-software).

You can manually download the GPS ephemeris data file from the Sony servers and move it onto the memory as described [here](https://blog.brixandersen.dk/2010/04/02/downloading-sony-gps-assist-data-using-perl/).

## Golang Based Solution
I've been playing around with GoLang lately so built a small CLI tool that download the GPS ephemeris file and loads it onto the camera: https://github.com/diarmuidie/assistme
