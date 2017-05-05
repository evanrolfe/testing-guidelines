We try to follow the rules for testing set out in [this](https://www.youtube.com/watch?v=URSWYvyc42M) talk.
Each method we test can be grouped into these two categories:

- **Queries**: methods which return data but have no side effects
- **Commands**: methods which have side effects on objects in our code base

Similarly each method can also be grouped into **Incoming** and **Outgoing** methods. Which of these groups a method belongs to depends on the perspective of the test. An incoming method is a method which is being directly tested by an rspec test while an outgoing method is a method that gets called on an external object by the incoming method. So in [this](https://github.com/TomBellCentegra/Centega-Stock/wiki/Unit-Testing-Guidelines#outgoing-command-messages) example, `Gear#set_cog` is the incoming method while `MyObserver#changed` is the outgoing method.

These rules are summarised in this table and more detailed notes for each testing situation are given below.

[[https://github.com/evanrolfe/testing-guidelines/blob/master/assets/unit_testing_grid.png]]

##Incoming Query Messages:

Example ruby class:
```ruby
  class Wheel
    attr_reader :rim, :tire

    def initialize(rim, tire)
      @rim = rim
      @tire = tire
    end

    def diameter
      rim + (tire * 2)
    end
  end
```

Example rspec test:
```ruby
describe '#diameter' do
  let(:wheel) { Wheel.new(3, 5) }

  subject { wheel.diameter }

  it { is_expected.to eq(13) }
end
```

- "assert on the result"
- "sight along the edges of the space capsules"
- "test the interface and not the implementation"

##Incoming Command Messages:

Example ruby class:
```ruby
  class Gear
    attr_reader :chainring, :cog, :wheel

    def initialize(chainring, cog, wheel)
      @chainring = chainring
      @cog = cog
      @wheel = wheel
    end

    def set_cog(new_cog)
      @cog = new_cog
    end
  end
```

Example rspec test:
```ruby
describe '#set_cog' do
  let(:wheel) { Wheel.new(3, 5) }
  let(:gear) { Gear.new(1, 3, wheel) }

  subject! { gear.set_cog(4) }

  it { expect(gear.cog).to eq(4) }
end
```
- "Test incoming command messages by making assertions about direct public side effects"

##Sent-to-self Query Messages:

Example ruby class:
```ruby
  class Gear
    attr_reader :chainring, :cog, :wheel

    def initialize(chainring, cog, wheel)
      @chainring = chainring
      @cog = cog
      @wheel = wheel
    end

    def gear_inches
      ratio * wheel.diameter
    end

    private

    def ratio
      chainring / cog.to_f
    end
  end
```

- do not test private method `ratio` as it is redundant and makes the test too brittle
- "break rule if it saves $$$ during development"

##Sent-To-Self Command Messages:
  - Same rules for sent-to-self query message, do not test.

##Outgoing Query Messages:

Example ruby class:
```ruby
  class Gear
    attr_reader :chainring, :cog, :wheel

    def initialize(chainring, cog, wheel)
      @chainring = chainring
      @cog = cog
      @wheel = wheel
    end

    def gear_inches
      ratio * wheel.diameter
    end

    private

    def ratio
      chainring / cog.to_f
    end
  end
```
- Do not make assertions on their value
- Do not expect them to be sent

##Outgoing Command Messages:

Example ruby class:
```ruby
  class Gear
    attr_reader :chainring, :cog, :wheel, :observer

    def initialize(chainring, cog, wheel, observer)
      @chainring = chainring
      @cog = cog
      @wheel = wheel
      @observer = observer
    end

    def set_cog(new_cog)
      @cog = new_cog
      observer.changed(new_cog)
      @cog
    end
  end
```

Example rspec test:
```ruby
describe '#set_cog' do
  let(:observer) { double(MyObserver) }
  let(:wheel) { Wheel.new(3, 5) }
  let(:gear) { Gear.new(1, 3, wheel) }

  before do
    allow(observer).to receive(:changed)
  end

  subject! { gear.set_cog(4) }

  it { expect(gear.cog).to eq(4) }
  it { expect(observer).to have_received(:changed).with(4) }
end
```
- Mock & expect outgoing command messages to be sent
