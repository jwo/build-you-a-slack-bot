## A What bot?


* SlackBot (n), a Bot for Slack
* Slack (n): a chat program, like IRC but for-profit and also Free and extensible
* Bot (n): Automated responder, preferably with fun to give illusion of AI

---
## Is Slack being used on the reg?

* 34 out of 35 Ruby devs at HoustonRB used it
* Only 10 moved from a previous chat program to Slack

---
## Slack Growth

![usage](/img/slack-growth.jpg)
---
## Slack is replacing email
---
## Slack is replacing texts
---
## Slack is replacing IRC
---
## Zapier
### You can Send everything to slack
---
Twitter Mentions to Slack
![img](/img/slack-twitter.png)
---

### other ideas

* Stripe Purchases
* Email to support
* Honeybadger issue
* GitHub issues


---
## If Zapier can do it, what can we do?

---
## Restated: How can we, as developers, use this for /good|evil/

---

## Things dorton and I have done:

1. Get current fantasy sports scores
1. Check sports rankings of teams
1. Connect to his BBQ tracking website and get current status of the brisket
1. Insult our friend when he said ScoreBot was stupid

---
## How Does this magic work?

1. A program sits and watches slack
1. It parses each message, decides if it should reply

---
## Sooooooo

1. No difficult to configure callbacks
1. It can run in dev mode easily
1. It runs on Free Slack
1. There's a gem for that

---
## Gem:

> gem 'slack-ruby-bot'

or

> gem 'slack-ruby-client'

---

## Which to use?

We'll use `slack-ruby-bot`, which wraps `slack-ruby-client`

---

## Hello World

Create a new team: https://slack.com/create

![configure](/img/configure.png)

---
## Hello World

Choose New Bot
![new-bot](/img/new-bot.png)

---
# Code and Stuff

---
## Hiiiiiiii World

![img](/img/rubybot-hi.gif)

---
## Hiiiiiiii World

```
require 'slack-ruby-bot'

module RubyBot
  class App < SlackRubyBot::App
  end

  class Hiiii < SlackRubyBot::Commands::Base
    command 'hi' do |client, data, _match|
      client.message text: 'hiiiiiiiiiiii', channel: data.channel
    end
  end
end

RubyBot::App.instance.run
```

---
## Wolfram Alpha

![img](/img/rubybot-wolfram.gif)

---

## Wolfram Alpha


```
class WolframSearch < SlackRubyBot::Commands::Base

    command 'wolf' do |client, data, _match|

      result = Wolfram::Query.new(_match[:expression]).fetch
      hash = Wolfram::HashPresenter.new(result).to_hash
      formatted_results = # snip
      client.message text: result, channel: data.channel

    end
  end
```

---
### Scrape Sites

Scraping Sites works very well.

![img](/img/rubybot-image.gif)


---

### Scrape Sites

Scraping Sites works very well.

```
class Image < SlackRubyBot::Commands::Base

  command 'imageme' do |client, data, _match|


    Mechanize.new { |agent|
      agent.user_agent_alias = 'Mac Safari'
    }
    .get("http://theironyard.com/about/team/")
    .search(".bio h2").select{|tag| tag.text =~ /#{_match[:expression]}/i}.map do |tag|
      {
        name: tag.text,
        src: "http://theironyard.com/" + tag.parent.parent.search("img").attr("src").value
      }
    end
    .each do |args|
      client.message text: args[:name], channel: data.channel
      client.message text: args[:src], channel: data.channel
    end


  end
end
```


---
### Insults are Funny

![img](/img/rubybot-insult.gif)

---
### Insults are Funny

```
class Insults < SlackRubyBot::Commands::Base

  command 'insult' do |client, data, _match|

    result = JSON.parse Http.get('http://quandyfactory.com/insult/json').body
    insult = result['insult']
    client.message text: "Hey #{_match[:expression]}, #{insult}", channel: data.channel

  end

end
```

---
## Screenshot of pages

---
<div class="wistia_responsive_padding" style="padding:73.54% 0 0 0;position:relative;"><div class="wistia_responsive_wrapper" style="height:100%;left:0;position:absolute;top:0;width:100%;"><iframe src="//fast.wistia.net/embed/iframe/rd4sgbb9p9?videoFoam=true" allowtransparency="true" frameborder="0" scrolling="no" class="wistia_embed" name="wistia_embed" allowfullscreen mozallowfullscreen webkitallowfullscreen oallowfullscreen msallowfullscreen width="100%" height="100%"></iframe></div></div>
<script src="//fast.wistia.net/assets/external/E-v1.js" async></script>

(or [gif](/img/rubybot-screenshot.gif))

---

### Screenshot of pages

```
class Screenshot < SlackRubyBot::Commands::Base

  command 'screenshot' do |client, data, _match|

    url = URI(_match[:expression].match(/\<(.*)\>/)[1]) rescue nil

    if url
      fetcher_object = Screencap::Fetcher.new(url.to_s)
      screenshot = fetcher_object.fetch width: 1700, height: 850
      imgur_client = Imgur2.new ENV['IMGUR_API']
      url = imgur_client.upload screenshot
      image_url = url["upload"]["links"]["original"]
      client.message text: image_url, channel: data.channel
    else
      client.message text: _match[:expression] + "isn't a URL, yo", channel: data.channel
    end
  end
end
```

---
## Other People's Good Ideas
---
## /healthcheck

![img](/img/healthcheck.png)

---
## Admin Slack instead of AdminPages

![img](/img/admin.png)

---
## Daily Standups

![img](/img/dailystandup.png)

---
## Merge Features (GitHub)

```
(merge)
bouzuya> hubot pr hitoridokusho/hibot #2
  hubot> #2 pull request 2
         hitoridokusho:master <- bouzuya:add-hubot-merge-pr
         https://github.com/hitoridokusho/hibot/pull/2
         OK ? [yes/no]
bouzuya> yes
  hubot> merged: hitoridokusho/hibot#2 : Pull Request successfully merged
```

(see https://github.com/bouzuya/hubot-pr)

---
# Lessons Learned

---

## Formatting Responses
You do all the text formatting. `\n` is your friend.

```
hash.fetch(:pods, {}).each do |key, values|
  next if values.join("") == ""
  result << "\n" + key + "\n"
  result << values.join("\n")
end
```
---
## You down with OPC

Other People's Code, like APIs and Webpages were likely not thought of as
needing to be friendly for YOU, a bot, to parse.

So they work, but you has lots of explaining to do.

---
# This RubyBot Code
---
## Tech Used

```
gem 'slack-ruby-bot'
gem 'dotenv'
gem 'wolfram'
gem 'http'
gem 'mechanize'
gem 'screencap'
gem 'imgur2'
```

---
## Where to Find

## [github.com/jwo/rubybot](https://github.com/jwo/rubybot)
