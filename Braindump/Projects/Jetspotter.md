---
tags:
  - projects
---

# Overview

[Jetspotter](https://github.com/vvanouytsel/jetspotter/) is a project that I created in order to, as the name implies, spot jets. The use case was simple. I have a small passion for fighter jets and I have been reflecting this on my children. Since I live in an area where fighter jets regularly fly over, it caused lots of fun moments where together with my children I run to the window to attempt to spot the aircraft. Unfortunately, as soon as you hear the jet, it already passed. Since I couldn't monitor [ADS-B Exchange](https://globe.adsbexchange.com/)the entire day, I needed something that could alert me whenever an aircraft was inbound to my location.

To solve that, I started developing [Jetspotter](https://github.com/vvanouytsel/jetspotter/). The application fetches its data from an API endpoint. You specify your coordinates, a list of aircraft types that you are interested in and and one of the supported notification webhooks and you will receive a notification whenever your jets has been spotted.

Now I can tell my children to have a look outside, because the A400 Atlas will be flying over house in about 60 seconds.

# The journey

I am always eager to learn something new and I wanted to get some hands on experience on developing something. However, I always lose interest if the thing I am building has no benefit to me. What is the point in coding a calculator since it already exists.

This time it was different, I had a use-case where I couldn't find a (free) solution for. As a DevOps Engineer the two languages I was most interested in were Python and Golang. Since I already had some experience with Python, I decided to use Golang instead.

## Getting started

I started simple. A golang application that you manually run. Once started, it will fetch an API endpoint to get a list of aircraft in your vicinity. During startup you pass your coordinates and a range. The API will provide a list of aircraft within range of your specified coordinate. Some parsing was done to print out interesting information such as callsign, aircraft type and altitude.

## Sending a notification

The next step was to add a notification to Slack. Since the parsing was already in place, this was rather simple. After Slack I decided to add support for Discord as well. Later on, I made notifications more generic so that they were easily expandable. Support for [Gotify](https://gotify.net/) and [Ntfy](https://ntfy.sh/) were also added. We even got a [mention](https://github.com/binwiederhier/ntfy/blob/main/docs/integrations.md) on the Ntfy integrations page.

At this point, you could run the application with its configuration parameters and receive a notification in one of the supported platforms.

## From cronjob to service

I had a VM in Hetzner that I used to deploy the application to. Using Ansible, a cronjob was created that ran the application every minute. This meant that whenever an aircraft that I was interested in (I filtered on military aircraft) a notification was send. This was awesome because it solved my use-case.

However, one of the drawbacks was that there was no state. This meant that if an aircraft kept flying in your vicinity, you would get a notification of the same aircraft. This let to notification fatigue and me often ignoring (or muting) my notifications.

In order to fix that, I needed to keep some kind of state of aircraft that were already spotted. Instead of using a database, I decided to keep it simple and to keep the state in memory. However, since my application would basically run to completion that was not possible, unless the entire application was rewritten.

This was the first major rewrite. Proper structs were created and the application did now run as a service. The trigger mechanism that was previously handled by cron, was now baked into the application itself.

This itself had a lot of benefits. I could finally keep track of state and only send notifications of aircraft that were not spotted before, reducing the alert fatigue. At this point I also decide to containerize the application and create a helm chart. Since I already had a k3s cluster running on my Hetzner VM, I used Ansible to deploy it as well.

## Setting up the documentation

I wanted to have some proper documentation instead of using the README file. I decided to go for [MkDocs](https://vvanouytsel.github.io/jetspotter/) as it was really simple to use and also quite beautiful in my opinion.

Hosting the documentation was also quite simple. If the GitHub repository is public, you can simply host it via GitHub Pages. The [documentation](https://vvanouytsel.github.io/jetspotter/) was deployed as part of the CI process that creates a release with every commit.

I wanted to document available configuration settings, but I was a bit scared of documentation drift. Often times documentation does not get updated whenever the code changes. In order to prevent that from happening, I decided to keep the documentation of the configuration part of the code.

I did this by combining `go doc` in a [Makefile](https://github.com/vvanouytsel/jetspotter/blob/main/Makefile). That way I could run `make doc` to automatically generate a snippet that I would include in the [configuration](https://vvanouytsel.github.io/jetspotter/configuration/) documentation page.

## I can't see any aircraft

"Cool, an aircraft is spotted at 20 000 feet. Damn, I can't see it, there are clouds everywhere."

I am lazy. Often times that is a bad habit, but in our field that is actually a good thing. Instead of looking through the window and see if there are any clouds, I could just have my application fetch data from a weather service instead.

I used [open-meteo](https://open-meteo.com/)to fetch weather data. Based on the data of the endpoint and the altitude of the spotted aircraft a cloud percentage was shown in the notification. If the percentage was close to 100%, you didn't have to bother to go outside, as the sky was full of clouds.

## Metrics are cool

As a DevOps Engineer, I often work with tools such as [Prometheus](https://prometheus.io/) and [Grafana](<https://grafana.trendminer.net/>. However, this is always from the point where the metrics are already available. I never attempted to create metrics myself. This looked like an interesting challenging so I decided to take it on.

There was an existing [library](https://prometheus.io/docs/guides/go-application/) to implement metrics in Go. The most difficult was to think about the metrics I wanted to keep. As the amount of metrics can quickly grow to millions if you are not careful.

I decided to track two metrics:
* `jetspotter_aircraft_spotted_total`: The total number of spotted aircraft.
* `jetspotter_aircraft_altitude_feet`: The altitude the jet is flying at.

With these two metrics I was able to keep track of how many of each type of aircraft I spotted. In order to properly use the altitude, I used a bucket. This meant that all aircraft flying between 0 and 2000 feet were added to a `<2000` bucket. With this I was able to see the distribution of altitude across the different types of aircraft.

[Dashboard example](https://vvanouytsel.github.io/jetspotter/dashboards)

## Extra filtering

After running the application for a couple of months I was really satisfied. I was tracking all military aircraft and knew that an A400 would always fly very low. However, the F16s flew very high, unless they were training "low" level flight. After a couple of months I didn't care anymore about jets that were flying high. I was only interested in aircraft that were flying very low.

To make that possible, filtering was added based on maximum allowed altitude. By default this was disabled (set to 0) but whenever specified, a notification would only be sent if the altitude of the spotted jet was below the defined maximum.

Now I get excited whenever my phone generates a notification sound. There is a 50% chance that either my wife sends me a message, or a fighter jet at low altitude is passing by. Both are equally thrilling.

# The future

In the future I would like to extend the application, though I have not set myself a timeline to do this. I often get big bursts of motivation to build stuff, but these also follow with periods of 'I want to do something completely different.'

## Building a front-end

Currently there is no front-end. I would love to learn more about front-end stuff by implementing a front-end in React.

This front-end would be used to:
* show aircraft in the vicinity
* show recently spotted aircraft
* show [[Metrics|dashboards]] directly inside the application
