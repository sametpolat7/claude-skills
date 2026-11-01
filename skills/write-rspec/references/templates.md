# Spec templates

- Service spec — through the public method, boundary-only stubbing
- Model spec — our logic only
- Request spec — full stack, the namespace's own auth
- New factory — lean base + traits

## Service spec — through the public method, boundary-only stubbing

```ruby
RSpec.describe Subscriptions::Renewal do
  let(:gateway) { instance_double(PaymentGateway) }

  def renew(subscription)
    described_class.new(subscription, gateway: gateway).call
  end

  describe '#call' do
    context 'when the charge succeeds' do
      it 'extends the subscription by one billing period' do
        subscription = create(:subscription, ends_on: Date.new(2026, 1, 31))
        allow(gateway).to receive(:charge).and_return(true)

        renew(subscription)

        expect(subscription.reload.ends_on).to eq(Date.new(2026, 2, 28))
      end
    end

    context 'when the charge is declined' do
      it 'marks the subscription past due and keeps its end date' do
        subscription = create(:subscription, ends_on: Date.new(2026, 1, 31))
        allow(gateway).to receive(:charge).and_return(false)

        renew(subscription)

        expect(subscription.reload).to have_attributes(status: 'past_due', ends_on: Date.new(2026, 1, 31))
      end
    end
  end
end
```

Assert persistence with `change`:

```ruby
expect { renew(subscription) }.to change(Invoice, :count).by(1)
```

## Model spec — our logic only

```ruby
RSpec.describe Subscription, type: :model do
  describe 'validations' do
    subject { build(:subscription) }

    it { is_expected.to validate_uniqueness_of(:external_id).scoped_to(:account_id) }
  end

  describe '#renewable?' do
    it 'is true for an active subscription that has reached its end date' do
      subscription = build(:subscription, ends_on: Date.current)

      expect(subscription).to be_renewable
    end

    it 'is false for a cancelled subscription, even past its end date' do
      subscription = build(:subscription, :cancelled, ends_on: 1.day.ago)

      expect(subscription).not_to be_renewable
    end
  end
end
```

## Request spec — full stack, the namespace's own auth

This example authenticates with a Devise session. Confirm route helpers with
`bin/rails routes -g <name>` before relying on them.

```ruby
RSpec.describe 'Admin::Reports', type: :request do
  include Devise::Test::IntegrationHelpers

  describe 'GET /admin/reports' do
    it 'lists reports for an admin' do
      sign_in create(:user, :admin)
      create(:report, title: 'Monthly revenue')

      get admin_reports_path, as: :json

      expect(response).to have_http_status(:ok)
      expect(response.parsed_body['data']).to contain_exactly(a_hash_including('title' => 'Monthly revenue'))
    end

    it 'turns away a signed-in user who is not an admin' do
      sign_in create(:user)

      get admin_reports_path, as: :json

      expect(response).to have_http_status(:forbidden)
    end
  end
end
```

## New factory — lean base + traits

```ruby
FactoryBot.define do
  factory :subscription do
    account
    sequence(:external_id) { |n| "sub-#{n}" }
    ends_on { Faker::Date.forward(days: 30) }

    trait :cancelled do
      status { 'cancelled' }
    end

    trait :past_due do
      status { 'past_due' }
    end
  end
end
```
