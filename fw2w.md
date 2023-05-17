# Using spies in rspec - 2023-05-14 23:33:45.402920157 -0300 -03 m=+74.642838018

Using spies, mocks and stubs in #rspec

```ruby
class Thermostat
  def initialize(thermometer:,furnace:)
    @thermometer = thermometer
    @furnace     = furnace
  end

  def check_temperature
    temp = @thermometer.temp_f
    if temp <= 67
      @furnace.turn_on or fail "Furnace could not be lit"
    end
  end
end

RSpec.describe Thermostat do
  it "turns on the furnace when the temperature is too low" do
    thermometer = double(temp_f: 67)
    furnace     = double(turn_on: true)
    thermostat  = Thermostat.new(thermometer: thermometer, furnace: furnace)

    expect(furnace).to receive(:turn_on)
    thermostat.check_temperature
  end

  it "leaves the furnace when the temperature is comfortable" do
    thermometer = double(temp_f: 72)
    furnace     = double
    thermostat  = Thermostat.new(thermometer: thermometer, furnace: furnace)

    expect(furnace).to_not receive(:turn_on)
    thermostat.check_temperature
  end
end
```

**Usar o to receive quebra a regra de 'Arrange, Act, Assert' no TDD. Para isso é possivel transformar o comportamento do double em um comportamento de spy transformando ele em um `have_received`**

Para fazer essa mudança, é preciso alterar o comportamento do double para aceitar o método `have_received`

```ruby
class Thermostat
  def initialize(thermometer:,furnace:)
    @thermometer = thermometer
    @furnace     = furnace
  end

  def check_temperature
    temp = @thermometer.temp_f
    if temp <= 67
      @furnace.turn_on or fail "Furnace could not be lit"
    end
  end
end

RSpec.describe Thermostat do
  it "turns on the furnace when the temperature is too low" do
    thermometer = double(temp_f: 67)
    furnace     = double.as_null_object # same as spy
    thermostat  = Thermostat.new(thermometer: thermometer, furnace: furnace)

    thermostat.check_temperature
    expect(furnace).to have_received(:turn_on)
  end

  it "turns on the furnace when the temperature is too low" do
    thermometer = double(temp_f: 67)
    furnace     = double(turn_on: true) # same as spy
    thermostat  = Thermostat.new(thermometer: thermometer, furnace: furnace)

    thermostat.check_temperature
    expect(furnace).to have_received(:turn_on)
  end

  it "leaves the furnace when the temperature is comfortable" do
    thermometer = double(temp_f: 72)
    furnace     = spy # same as double.as_null_object
    thermostat  = Thermostat.new(thermometer: thermometer, furnace: furnace)

    thermostat.check_temperature
    expect(furnace).not_to have_received(:turn_on)
  end
end
```



