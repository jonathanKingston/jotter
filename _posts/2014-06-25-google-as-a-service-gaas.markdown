---
layout: post
title: "Google as a Service (GaaS)"
date: 2014-06-25 08:50
updated: 2015-03-13 00:00
comments: true
categories: [google, cloud, services, tech, development]
---
Google appears to be pushing further into the open internet platform in a land grab bid against closed source providers. As other articles are suggesting, Google is pushing the 'Android everywhere' idea however I am seeing this more as a two pronged approach with an 'Internet everywhere' approach as the main driver for Google's other profitable offerings.

This article covers some of what we might see at [Google I/O](https://www.google.com/events/io) but also some of which could even be for next year. Google seem to want to close the gap between the open internet and native applications, retrofitting much of the tech that is provided in native applications into web based offerings. I would likely argue this is to win over some of the adopters of [Apple's new Swift offering](https://developer.apple.com/swift/) but also backed by consumer to corporate offerings that markets using "Google for Everything"

The latest Google I/O talks seem to be very much focused on the developers this time: probably to counter the new Swift language by Apple but also to push more of Google's platform to the early adopters. So, we are seeing less consumer and business announcements this year and instead a focus on a long game of developer adoptions for an array of new Google services.

<!-- more -->

# Backing the open internet
Google is investing further time and money into being an open platform which gives developers the confidence that the tech they integrate with should hopefully stay longer than heavy investment in Google APIs. They are also spending a lot of time bridging the open internet divide with Android as it brings further contention to the closed source application frameworks by bringing a near-native experience to all of the web. I think they are doing this mostly to further open up integration to pushing their main profit making storage, server, applications and devices.

## Investment into Universal 2nd Factor payment mechanisms
Google has spent a lot of its efforts in writing and developing a strong authentication called [U2F](https://sites.google.com/site/oauthgoog/gnubby) method between two factor authentication and web applications. This has now been opened into an [alliance](http://fidoalliance.org/) which looks to be opening ID management and authentication online.

## Polymer
[Polymer](http://www.polymer-project.org) is a patch for the future of the net, a patch that was made by Google
 
[Active web updates for changes](http://www.polymer-project.org/docs/polymer/node_bind.html)

Polymer gives developers the ability to write out an application and give a modern unified look and feel to their applications cross browser. Features such as reusable templates in the front end, with the ability to bind changes on the back-end to these templates (Think Meteor but native).

[Google I/O talk on Polymer](https://www.google.com/events/io/schedule/session/de22e147-07b6-e311-8491-00155d5066d7)


## Native app feel for mobile web
Google is now pushing a new feature on Android '[Install to homescreen](https://developer.chrome.com/multidevice/android/installtohomescreen)' which is implemented with `mobile-web-app-capable` as a meta extension.


`brand-color` - [this feature](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/nzRY-h_-_ig/KR3XWn73tDoJ) is to add a hint to mobile tech on what colour represents the site; the browser specific implementations in Apple and IE pick tile colours and status bar colours.
This thread also mentions showing at Google I/O which likely shows further push into making the mobile web native in this Google I/O.

## Geofencing
Geofencing is the ability to monitor when the user enters a certain geographical area and provide notifications based upon it. This is a further step to bringing mobile based tech to websites of any nature.

[Geofencing bug](https://code.google.com/p/chromium/issues/detail?id=383125&q=getRegisteredRegions&colspec=ID%20Pri%20M%20Iteration%20ReleaseBlock%20Cr%20Status%20Owner%20Summary%20OS%20Modified)

## Harmony between SVG and HTML
[Some effort](https://code.google.com/p/chromium/issues/detail?id=243882) has been made by Google focused on changing over SVG rendering into the same as HTML.

This looks like it is to pave the way for the following new specifications:

- [Geometry interfaces module](http://dev.w3.org/fxtf/geometry/)
- [SVG integration](http://www.w3.org/TR/svg-integration/)

**What this likely means:**
SVG features and customisations become closer to HTML, giving a native application user experience to the web.

##Investments in web tech
Google are building apps, platforms and guides which all promote further more of Google's latest technologies they are preferring at the moment.

[Web fundamentals site](https://developers.google.com/web/fundamentals/)
Google is providing yet more guides in building web sites; the latest is advice and dev support in multiple device technologies.

They have published a [starter kit](https://developers.google.com/web/starter-kit/) which promotes some of Google's best practices and development tools. Interestingly this appears to be backing the [appmanifest](http://www.w3.org/TR/appmanifest/) which in the particular form it is in with a `.webapp` extension was designed for the [Firefox marketplace](https://marketplace.firefox.com/developers/?src=nav_logo).


### Angular and friends
More support into providing Google services and ideas into the web stack. For example Angular promotes web components and other new technologies mentioned in this article.

# Marketing Google as a Service

As well as ramping up marketing new cloud based services to developers - [With fancy visual developer pages](https://developers.google.com/drive/) and Google are also advancing their offerings in cloud which are also marketing the ecosytem too.

## Google domains
Google is going to be a [domain provider](http://domains.google.com/about/) which is another promotion for apps and hosting services that they provide. Linking together Google mail and other quick web app tools they already have with domains is likely to give the consumers and companies a much quicker way to get online.

[Google talk on the future of domains](https://www.google.com/events/io/schedule/session/22ce27dc-7cbf-e311-b297-00155d5066d7) - this will likely talk through Google entering into the domain market, after all they have secretly been backing some of the [new GTLD's](http://newgtlds.icann.org/en/) with its new [Charleston road registry which has become public recently](https://www.google.com/registry/) including other extensions like: [.dev](https://gtldresult.icann.org/application-result/applicationstatus/applicationdetails/1339), [.family](https://gtldresult.icann.org/application-result/applicationstatus/applicationdetails/499), [.android](https://gtldresult.icann.org/applicationstatus/applicationdetails/519) and [.page](https://gtldresult.icann.org/application-result/applicationstatus/applicationdetails/511).

Future expansions could lead to one click installation of analytics, hosting, cloud applications.

## Enterprise offerings

Google is making waves in changes for content distributors from becoming its own [ISP](https://fiber.google.com/about/) to building corporate friends with the likes of [Netflix by lobbying with them for an open internet](https://www.google.com/takeaction/) against companies like Comcast and the FCC.

### ISP content distribution
Google's clear support for distribution of web media is a focus for them as a whole company:

- [Making your cloud apps Google-fast](https://www.google.com/events/io/schedule/session/0486a8f4-4acb-e311-b297-00155d5066d7) - providing new companies the architecture and tools to be as good as Google for performance is likely to save companies a lot of money.
- Advancement into web encodings like [WebP](https://developers.google.com/speed/webp/?csw=1) and [WebM](http://www.webmproject.org/) put trust in companies that Google has their back, also the storage and network savings will likely be massive.
- [Fiber to content](http://googlefiberblog.blogspot.co.uk/2014/05/minimizing-buffering.html) - through colocation of content the service improvements would be huge.
- [Encrypted media extensions support](https://dvcs.w3.org/hg/html-media/raw-file/tip/encrypted-media/encrypted-media.html) - Unfortunately media companies being what they are with the lack of encrypted media on the web content providers would refuse to distribute content over the open web. Even on the outset this seems a step backwards for web standards actually; it promotes the web as a full stack application with no proprietary plugins. By standardising this approach web authors; browsers and content distributors won't need to bring their own tech to the table.
- Adding improvements in their cloud platform to compete in the market with: [Investment in Docker for scaling of applications](https://developers.google.com/compute/docs/containers) and [New import from AWS tool](http://techcrunch.com/2014/06/19/googles-new-cloud-import-tool-makes-switching-from-aws-easier/) giving corporates less trouble in designing for scale makes Google a great AWS competitor.

### Universal Communications
Google's technologies appear to be aligning slowly whereby I can see an always-on approach to communication going through the best path of communication:

- [Partnership with Twilio](https://www.twilio.com/blog/2014/06/twilio-and-google-twilio-cx-for-chromebooks-nt.html) - Leveraging [WebRTC](http://www.webrtc.org/) which is another open Google backed service
- [Improvements of hangouts](https://plus.google.com/+googleplus/posts/8irsMuNR3ZK) merged conversations with SMS chats is the first step to providing a communication tool where the user doesn't need to worry about which of the N contact mechanisms to take - the software can decide and pick the most convenient for the recipient.
- ['Glass at work'](https://developers.google.com/glass/distribute/glass-at-work) partnerships - likely to roll out tech for down time for employees; commuting etc
- [Google casting](https://developers.google.com/cast/) - Which is bringing the ability to share media cross device and could be modified to open up universal communications for video conferencing.
- Google cloud messaging on more devices with [Chrome based devices with native gcm](https://developer.chrome.com/apps/cloudMessaging) - This was announced last year but has only just dropped into [Chrome 35](http://googlechromereleases.blogspot.co.uk/2014/06/stable-channel-update.html) as unified API. Allowing sending and receiving Google cloud messages through chrome applications. This means developers should be able to communicate with all apps soon. Internally [Google's cloud print](http://www.google.co.uk/cloudprint/learn/) used GCM; imagine the potential with Google opening this up to all developers.

# Conclusion

Google appears to be playing a very long game with helping developers; which I think will likely pay off for them in advertising and distributing their tech services and products. By offering up a complete web solution to most in the market, above and beyond what its competitors can it will likely attract a wide range of consumers to companies. Google is betting on whole companies riding on their platform from handset to servers.


