# Testing (WIP)

This page is woefully incomplete because we need your help to finish it.

In the future, this page will be home to the definitive guide to testing StimulusReflex. Until then, we're doing our best!

After many conversations and a few threats of potential future action, sometimes the only way to get started is to start. The stucture, order and content best suited to the topic of testing is still very much an open conversation.

Please do drop by [#docs](https://discord.gg/kCnM5Zfvau) on the StimulusReflex Discord and offer your best ideas. Please **don't** open documentation PRs on Github, as we can't accept them for technical reasons.

### &#x20;<a href="#test-environment-setup" id="test-environment-setup"></a>

Setting up your test environment to run StimulusReflex is very similar to what you probably already have running in development. Please verify that Reflexes are working in development before troubleshooting your test environment.

Here is a checklist of what needs to be enabled, much of which is borrowed from the development environment setup:

Install [Redis](https://redis.io/download). Make sure that it's running and accessible to the Rails project and then include connectivity gems:

gem "redis", ">= 4.0", :require => \["redis", "redis/connection/hiredis"]

To setup your Rails credentials for the test environment and link to Redis, run `rails credentials:edit --environment test` and add the following:

redis\_url: redis://localhost:6379/0

Configure ActionCable to use your Redis instance:

url: <%= Rails.application.credentials.redis\_url %>

channel\_prefix: your\_app\_test

Configure your cache store and turn on ActionController caching:

config/environments/test.rb

config.action\_controller.perform\_caching = true

config.cache\_store = :redis\_cache\_store, {driver: :hiredis, url: Rails.application.credentials.redis\_url}

### &#x20;<a href="#resources" id="resources"></a>

### &#x20;<a href="#actioncable-testing-guide" id="actioncable-testing-guide"></a>

### &#x20;<a href="#stimulus_reflex_testing-gem" id="stimulus_reflex_testing-gem"></a>

Our friends at Podia released [stimulus\_reflex\_testing](https://github.com/podia/stimulus\_reflex\_testing), which provides some helpers for unit testing your Reflex classes.

### &#x20;<a href="#open-questions" id="open-questions"></a>

How do you run the StimulusReflex tests on the server? How do you run them on the client?

Where do we need more coverage?
