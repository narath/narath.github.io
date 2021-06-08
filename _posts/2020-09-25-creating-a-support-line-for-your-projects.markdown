---
layout: post
title: "Creating a support line for your projects"
date: "2020-09-25 10:35:36 -0400"
---

I know I like it a lot when there is fast access to a person if I need help. Sure, it's not always scalable, but for many agile projects, you do want your users to be able to quickly talk to you. 

I was launching a new service and I wanted to make sure the users could quickly talk to me with any problems they might have. But I also needed to set some limits on my time, but still help them get their message to me reliably. I also did not want to use my personal number, since (1) others might also help support the project, (2) my ability to support the project might end.

Many people solve this problem, by either (1) just giving their phone number, or (2) not giving a phone number, just leaving a generic email or website address.

But there is lots of technology to help facilitate human connections while helping keep humane limits. Here's how I did it.

Used Callcentric to get a new phone number.

Create an extension for this number. This creates a SIP account for this.

Get a VOIP softphone. I used Bria mobile. Add the SIP address you created for your extension.

I wanted to give my users the chance to be able to just leave a message, or to be able to try to reach us immediately. To do this you have 2 options:

1. Setup an IVR. This will let you let the user just leave a voice message straight away, or ring our support group.
  * setup an interactive voice response system (IVR) that let them choose to leave a message, or ring our support group (me and another person)
  * setup a Call Treatment, that directs calls to that number to the IVR.
    * the call treatment lets me choose hours when this applies, so I can set weekend hours to go straight to voice mail.
2. Just setup a voice mail and a call treatment
  * For your extension, setup a custom voicemail (have it mail it to you, and you can use your phone's text email to get notified by sms)
  * Add a call treatment. I used Hunting -> extension (my Bria VOIP line), then my partners cell, then custom voicemail

Make sure you setup voicemail in callcentric. Extensions > (Choose Extension) > Voice Mail > Custom Voicemail for this extension.

So what were the total costs for this:

| Cost   | Item                    |
| ------ | ----------------------- |
| $ 3.95 | setup fee               |
| $ 1.95 | monthly fee             |
| $ 0.99 | for Bria mobile monthly |

So `$4.00 + $2.95` a month. Not bad, for what I hope will be a much better experience for my users, and still a shared load, and humane system of communication for me.

# Related

## How to record, convert and upload messages

I just use Voice Memo on my iphone to record the message. Then I use ffmpeg to convert it to an mp3.

```sh
brew install ffmpeg
ffmpeg -i memo.m4a memo.mp3
```



