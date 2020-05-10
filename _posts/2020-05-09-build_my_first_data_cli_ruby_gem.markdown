---
layout: post
title:      "Build My First Data CLI Ruby Gem"
date:       2020-05-10 02:38:24 +0000
permalink:  build_my_first_data_cli_ruby_gem
---


It wasn't easy at first building my first CLI data Gem because I came across a lot of errors, code, testing
but I never gave up from keep try to build, I had a lot of project, I scraped away (trast), till I decide to build a weather gem
It wasn't not that hard to do a weather gem, but I did do a bit of research how to build it, like Set up the weather API account in www.openweathermap.org which the account is free after register, but im going tell you what I done first, build a simple weather CLI using plain Ruby with the help of [bundler](http://https://bundler.io/) 

First of all let’s create a new Gem project using: `bundle gem weather --test=minitest`

Next we need Customise the gemspec to.

```
spec.summary = "simple weather CLI"
spec.description = "Simple weather CLI using OpenWeatherMap API"
spec.homepage = "http://idonthaveanybutyoumayhaveone.io"

spec.default_executable = "weather"
```

Then  next we going Set up the weather API account.

Go to https://openweathermap.org/ and register for a free  account to get an app ID

I’ve obtained an app ID and set it as environment variable `WEATHER_APP_ID` so that we don’t commit it to the source control system. Hour gem is going to read it from the global `ENV` variable.

Using Test-driven development we design our weather client. We start by designing a `Weather::Gateway` that we will use to encapsulate the details of the API provider, so that if we decide to user another provider, our changes are constrained within this class.

```
require 'test_helper'

describe Weather do
  it 'has a version number' do
    refute_nil Weather::VERSION
  end
end

describe Weather::Gateway do
  before do
    app_id = ENV['WEATHER_APP_ID']
    abort "set WEATHER_APP_ID env variable" unless app_id
    @gateway = Weather::Gateway.new(app_id)
  end

  it 'gets forecast for New York' do
    samples = @gateway.forecast('New York')

    assert samples.any?
    samples.each do |sample|
      assert_respond_to sample, :date
      assert_respond_to sample, :description
      assert_respond_to sample, :humidity
      assert_respond_to sample, :temp_min
      assert_respond_to sample, :temp_max
    end
  end
end
```
Here is a basic implementation that satisfies the test. We do it in a file called `lib/weather.rb`

```
require 'net/http'
require 'json'
require 'weather/version'

module Weather
  Sample = Struct.new(:date, :temp_min, :temp_max, :humidity, :description)

  class Gateway
    ROOT_URL = 'http://api.openweathermap.org/data/2.5'

    def initialize(app_id)
      @app_id = app_id
    end

    def forecast(query)
      uri = forecast_uri_for(query)
      response = Net::HTTP.get_response(uri)

      if response.code == '200'
        sample_list_from_response(response)
      else
        raise response.body
      end
    end

    private 

    def sample_list_from_response(response)
      data = JSON.parse(response.body, symbolize_names: true)

      data[:list].map do |sample|
        sample_from_json(sample)
      end
    end

    def sample_from_json(json)
      Sample.new(
        Time.at(json[:dt]).strftime('%Y-%m-%d %H:%M'),
        json[:main][:temp_min],
        json[:main][:temp_max],
        json[:main][:humidity],
        json[:weather].first[:description]
      )
    end

    def forecast_uri_for(query)
      params = { 
        units: 'metric', 
        appid: @app_id,
        q: query 
      }
      uri = URI(ROOT_URL + '/forecast')
      uri.query = URI.encode_www_form(params)
      uri
    end
  end
end
```


It's Time to edit `exe/weather` to make use of the `Weather::Gateway` class and print out the results to the terminal.

when installing the gem on Windows, Rubygem will automatically create a ` weather.bat` file that runs the `exe/wather.rb` file using the currently active Ruby installation. now when ready to try this Gem out type in terminal the word weather and zipcode 31206, check out my pic----> [weather](https://ibb.co/MhFw8y4)
