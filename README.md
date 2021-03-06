# Clearbit Slack Notifier

Clean beautiful customer data. Now in Slack.

![alex test](https://cloud.githubusercontent.com/assets/739782/19399022/44627600-9204-11e6-9b80-447fb5685bad.png)

## Installation

Add to your application's Gemfile:

```ruby
gem 'clearbit-slack'
```

### Configuration

Add Clearbit and Slack config vars:

```ruby
# config/initializers/clearbit.rb
Clearbit.key = ENV['CLEARBIT_KEY']

Clearbit::Slack.configure do |config|
  config.slack_url = ENV['SLACK_URL']
  config.slack_channel = '#signups'
end
```

### Clearbit Key

Sign up for a [Free Trial](https://clearbit.com/) if you don't already have a Clearbit key.

## Notifications

### Parameters

| Field       | Description                                        |
| ----------- | -------------------------------------------------- |
| company     | Company data returned from Clearbit                |
| person      | Person data returned form Clearbit                 |
| message     | Used for deep link into an internal Admin/CRM      |
| email       | Used to augment message data when Person not found |
| full_name   | Used to augment message data when Person not found |

### Streaming API

Lookup email using the Clearbit streaming API and ping Slack channel:

```ruby
# app/jobs/signup_notification.rb
module APIHub
  module Jobs
    class SignupNotification
      include Sidekiq::Worker

      def perform(customer_id)
        customer = Customer.find!(customer_id)
        result = Clearbit::Enrichment.find(email: customer.email, given_name: customer.first_name, family_name: customer.last_name, stream: true)

        result.merge!(
          email: customer.email,
          full_name: "#{customer.first_name} #{customer.last_name}",
          message: "View details in <https://admin-panel.com/#{customer.token}|Admin Panel>",
        )

        Clearbit::Slack.ping(result)

        # Save Clearbit data to datastore
      end
    end
  end
end
```

### Webhooks

Receive a webhook with Clearbit data and ping Slack channel:

```ruby
# app/controllers/webhooks_controller.rb
class WebhooksController < ApplicationController
  def clearbit
    webhook = Clearbit::Webhook.new(env)
    customer = Customer.find!(webhook.webhook_id)
    result =  webhook.body

    result.merge!(
      email: customer.email,
      full_name: "#{customer.first_name} #{customer.last_name}",
      message: "View details in <https://admin-panel.com/#{customer.token}|Admin Panel>",
    )

    Clearbit::Slack.ping(result)

    # Save Clearbit data to datastore
  end
end
```

## Contributing

1. Fork it ( https://github.com/[my-github-username]/clearbit-slack/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
