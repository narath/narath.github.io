---
layout: post
title: "Enabling Cache-Control in Rails 6 Development"
date: "2021-06-08 14:26:47 -0400"
---

I'm working on using Rails as an API for an iOS mobile app. Swift's URLRequest supports [client side caching protocols](https://developer.apple.com/documentation/foundation/url_loading_system/accessing_cached_data) which is great, and can work beautifully with Rails if you can send the Cache-Control headers. Here's how I got it to work.

## In Rails:

You need to disable mini-profiler by commenting it out in your Gemfile. Unfortunately it will overwrite your headers, and I did not see how I could turn it off.

```rb
# gem 'rack-mini-profiler', '~> 2.0'
```

In your controller action, add `expires_in(time_in_seconds)`

```rb
  def index
    @recommendations = Treasure.recommendations(@credential.user)
    @result = { records: @recommendations, offset: "" }
    respond_to do |format|
      format.json { render json: @result }
    end
    expires_in 10.minutes
  end
  ```

## In Swift:

You can now just use the `.useProtocolCachePolicy` and it will use the Cache-Control directive information to control the cache.

```swift
var request: URLRequest

// setup method, endpoint, params etc

// Cache
if useCache {
  request.cachePolicy = .useProtocolCachePolicy
}
```

## Helpful Articles:

- [BigBinary on how to enable caching in development](https://bigbinary.com/blog/caching-in-development-environment-in-rails5)
- [Helpful HoneyBadger article describing caching including the Cache-Control terms](https://www.honeybadger.io/blog/http-caching-ruby-rails/)
- [Rails Guide on Caching](https://guides.rubyonrails.org/caching_with_rails.html) - in general be cautious with client side caching (and caching in general, it is hard to get just right
- [Heroku advocates a CDN which needs cache control - they talk about it here](https://devcenter.heroku.com/articles/http-caching-ruby-rails)
