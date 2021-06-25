---
title: Correctly Setting Up Production and Staging Environments with Swift Custom Flags
layout: post
date: "2021-06-25 06:00:00 -0500"
category: xcode
tags: xcode, mac, compiler
---

Today I had a lot of fun trying to get Xcode to correctly use my swift compilation flags so that I can properly differentiate between develop, staging and production environments. 

```swift
#if DEVELOP
        static let server_host = "..."
#elseif STAGING
        static let server_host = "..."
#elseif PRODUCTION
        static let server_host = "..."
#else
#error("You have not defined an environment for app credentials")
#endif
```

Here are the steps that do work

1. Add the appropriate build configurations to your app:
  - click on the app in the file list view
  - select the Project
  - under Info, use + to add new configurations. I am using:
    - Debug
    - Debug-Staging
    - Staging
    - Release
2. Now in Targets select your app again, find "Swift Compiler - Custom Flags"
  - for Debug builds, just adding it to the main line seems to work
  - for all other configurations, don't add it to the main line (it does not seem to get picked up here), but instead, click the (+) and add it to "Any Architecture | Any SDK"
  - Note: you don't need -DFLAG as identified in some posts

![This will work](/assets/images/swift-custom-flags/will_work.png)

## Gotcha: this looks like it would work, but it does not (at this time)

![This does not work](/assets/images/swift-custom-flags/wont_work.png)


