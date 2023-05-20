# Checklist to reduce factory usage - 2023-05-18 21:00:42.763525703 -0300 -03 m=+398.929874078

tags: #platform #testing #performance

## Checking if you have a factory cascade problem

1.  Using (https://test-prof.evilmartians.io/#/)\[test-prof] gem to check if you have a factory cascade problem
2.  Checking with `FPROF=1 bundle exec rspec spec/` if you have a factory cascade problem

## Fixing the factory cascade problem

1.  Duplicate the factory with a deprecated or valid version
2.  Move all associations from default factory to the deprecated or valid version
3.  For all failing specs, replace the factory with deprecated versio
4.  Manually replace usage of deprecated factory with traits

## References

```ruby
FactoryBot.define do
    factory :deprecated_session do
        event
        start_date { Date.tomorrow.at_midday }
        end_date { Date.tomorrow.at_midday + 1.hour }

        after(:create) do |session|
            create_list(:participant, 3, session: session)
        end
    end

    factory :session do
        event
        start_date { Date.tomorrow.at_midday }
        end_date { Date.tomorrow.at_midday + 1.hour }

        transient do
            participants_count { 1 }
        end

        trait :with_participants do
            after(:create) do |session, evaluator|
                create_list(:participant, evaluator.participants_count, session: session)
            end
        end
    end
end
```
