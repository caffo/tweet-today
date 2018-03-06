#! /usr/bin/env ruby

require "rubygems"
require "bundler"

Bundler.require

require "sendgrid-ruby"
include SendGrid

task default: :tweet

desc "Gets tweets"
task :tweet do
  raise "$TWITTER_KEY not set in ENV, cannot start." if ENV["TWITTER_KEY"].nil?
  raise "$TWITTER_SECRET not set in ENV, cannot start." if ENV["TWITTER_SECRET"].nil?
  raise "$TWITTER_LIST not set in ENV, cannot start." if ENV["TWITTER_LIST"].nil?

  client = Twitter::REST::Client.new do |config|
    config.consumer_key = ENV["TWITTER_KEY"]
    config.consumer_secret = ENV["TWITTER_SECRET"]
  end

  username, list_id = ENV["TWITTER_LIST"].split("/")

  messages = client.list_timeline(username, list_id, count: 1000).delete_if do |t|
    Chronic.parse("last night") > t.created_at
  end

  template = Tilt.new("mail.erb")
  html = template.render(nil, messages: messages,
                              oembeds: client.oembeds(messages, omit_script: true))

  if ENV["SENDGRID_API_KEY"]
    raise "$EMAIL_TO_ADDRESS not set in ENV, cannot email." if ENV["EMAIL_TO_ADDRESS"].nil?
    raise "$EMAIL_FROM_ADDRESS not set in ENV, cannot email." if ENV["EMAIL_FROM_ADDRESS"].nil?

    from    = Email.new(email: ENV["EMAIL_FROM_ADDRESS"])
    to      = Email.new(email: ENV["EMAIL_TO_ADDRESS"])
    content = Content.new(type: 'text/html', value: html)
    subject = "Tweet Today #{Time.now.strftime("%F")}",

    email    = Mail.new(from, subject, to, content)
    client   = SendGrid::API.new(api_key: ENV['SENDGRID_API_KEY'])
    response = client.client.mail._('send').post(request_body: email.to_json)

    p email.to_json
    p response
  else
    puts html
  end
end
