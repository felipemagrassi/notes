# Domain Services - 2023-05-20 19:45:23.487103098 -0300 -03 m=+12815.894120797

tags: #arkency #fleeting #services

Domain services are a special kind of service that is not part of the domain model. It is a
segment of domain logic that does not belong to any aggregate. Calculators, managers, policies...

```ruby

class InvoiceCalendar
  class Terms < Struct.new(
      :invoice_due_at,
      :credit_note,
      :release,
      :optional_reminder1,
      :optional_reminder2
    )
  end

  FREE_DAY = Proc.new do |country_code, date|
    date.saturday? ||
    date.sunday?   ||
    date.holiday?(country_code, :observed)
  end

  def initialize(free_day: FREE_DAY)
    @free_day = free_day
  end

  def call(current_date:, concert_date:, country_code:)
    invoice_due_at = invoice_due_at(country_code, current_date, concert_date)

    credit_note = add_working_days(country_code, invoice_due_at, 1)
    release = add_working_days(country_code, credit_note, 1)

    optional_reminder2 = subtract_working_days(country_code, invoice_due_at, 3)
    optional_reminder2 = nil unless current_date + 2 <= optional_reminder2

    optional_reminder1 = subtract_working_days(country_code, invoice_due_at, 7)
    optional_reminder1 = nil unless current_date + 2 <= optional_reminder1

    Terms.new(
      invoice_due_at,
      credit_note,
      release,
      optional_reminder1,
      optional_reminder2
    )
  end

  private

  def invoice_due_at(country_code, current_date, concert_date)
    due_date_candidate1 = add_working_days(country_code, current_date + 13, 1)
    due_date_candidate2 = subtract_working_days(country_code, concert_date, 3)
    [due_date_candidate1, due_date_candidate2].min
  end

  def subtract_working_days(country_code, date, days)
    days.times do
      loop do
        date -= 1
        break unless free_day?(country_code, date)
      end
    end
    date
  end

  def add_working_days(country_code, date, days)
    days.times do
      loop do
        date += 1 break unless free_day?(country_code, date)
      end
    end
    date
  end

  def free_day?(country_code, date)
    @free_day.call(country_code, date)
  end
end

```

This calculator is stateless and side-effect free. It just takes 3 values as input
and returns 5 values as output. It does not alter the state of any object.

It makes InvoiceCalendar easier to test.

```ruby

RSpec.describe InvoiceCalendar do
  specify "when concert is far from now, due at is 14 days later" do
    calendar = InvoiceCalendar.new(
      free_day: ->(country_code, date){ date.day % 3 > 0}
    )
    result = calendar.call(
      current_date: Date.new(2016, 1,  1),
      concert_date: Date.new(2016, 1, 25),
      country_code: :uk,
    )
    expect(result.invoice_due_at).to eq(Date.new(2016, 1, 15))
    expect(result.credit_note).to    eq(Date.new(2016, 1, 18))
    expect(result.release).to        eq(Date.new(2016, 1, 21))
  end
end

```
