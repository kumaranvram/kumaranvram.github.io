---
layout: post
title: "ZeroPush - Simple push notifications on iOS"
date: 2015-04-15 15:19
comments: true
categories: 
---
I was recently working on an iOS application which requires push notifications. It is a not-so-pleasurable experience for any developer to configure the backend of the application to send push notifications. The general reasons for this displeasure are due to device/token management on the server and worker processes on the server which should send the notifications without blocking the main thread.

I recently bumped upon [ZeroPush](http://zeropush.com) which relieves the developer of both of these pain points. The implementation of ZeroPush for push notifications was very easy both on the client and server side.
<!-- more -->
####Device/Token management:
ZeroPush takes care of registering the device to the Apple's Push notification. If you wish not to undergo the pain of token management _at_ _all_, using ZeroPush named channels where the user can listen to notifications can be created and listened to. This gives a breath of fresh air to the developers who loves to save a lot of time. 

####Push notification management:
On the server side, (in my case, a rails application in Heroku), integrating ZeroPush was easy with their gem. Once their gem is added, the notifications can be sent by creating a hash table with the necessary parameters:
```ruby
ZeroPush.notify({
	device_token: ['token1', 'token2'],
	alert: 'This is a notification',
	badge: 1,
	sound: "default",
	info: {
		'use': 'Anyother information that needs to be sent to the device along with the payload'
	}
})
```
[Yes, I still went with tokens than using channels :(] This makes a rest call to the ZeroPush server which takes care of the notifications. But for ZeroPush, I might have ended up writing SideKiq worker processes, backed by Redis which definitely is not as easy as this.

ZeroPush comes at a very small price (starting at $10 per month), but it is definitely much cheaper than installing SideKiq/Redis on Heroku and, not to mention, the effort involved.
