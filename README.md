# ConcurrentRails

![status](https://github.com/luizkowalski/concurrent_rails/actions/workflows/ruby.yml/badge.svg?branch=master)

Multithread is hard. [concurrent-ruby](https://github.com/ruby-concurrency/concurrent-ruby) did an amazing job
implementing the concepts of multithread in the Ruby world. The problem is that Rails doesn't play nice with it. Rails have a complex way of managing threads called Executor and concurrent-ruby (most specifically, [Future](https://github.com/ruby-concurrency/concurrent-ruby/blob/master/docs-source/future.md)) does not work seamlessly with it.

The goal of this gem is to provide a simple library that allows the developer to work with Futures without having to care about Rails's Executor and the whole pack of problems that come with it: autoload, thread pools, active record connections, etc.

## Usage

This library provides two classes that will help you run tasks in parallel: `ConcurrentRails::Future` and `ConcurrentRails::Multi`

### Future

`ConcurrentRails::Future` will execute your code in a separated thread and you can check the progress of it whenever you need. When the task is ready, you can access the result with `#result` function:

```ruby
irb(main):001:0> future = ConcurrentRails::Future.new do
  sleep(5) # Simulate a long running task
  42
end

# at this point, nothing has happened yet.

irb(main):002:0> future.execute

irb(main):003:0> future.state
=> :processing

# after 5 seconds
irb(main):004:0> future.state
=> :fulfilled

irb(main):005:0> future.value
=> 42
```

A task can also fail. In this case, the state of the future will be `rejected` and the exception can be accessed by invoking `reason`

```ruby
irb(main):001:1* future = ConcurrentRails::Future.new do
irb(main):002:1*   2 / 0
irb(main):003:0> end.execute

=> #<ConcurrentRails::Future...

irb(main):004:0> future.state
=> :rejected

irb(main):005:0> future.reason
=> #<ZeroDivisionError: divided by 0>
```

### Multi

`ConcurrentRails::Multi` will let you execute multiple tasks in parallel and aggregate the results of each task when they are done. `Multi` accepts an undefined number of `Proc`s.

```ruby
irb(main):001:1* multi = ConcurrentRails::Multi.enqueue(
irb(main):002:1*   -> { 42 },
irb(main):003:1*   -> { :multi_test }
irb(main):004:0> )

=> #<ConcurrentRails::Multi:0x00007fbc3f9ca3f8 @actions=[#<Proc:0x00007fbc3f9ca470..
irb(main):005:0> multi.complete?
=> true

irb(main):006:0> multi.compute
=> [42, :multi_test]
```

Given the fact that you can send any number of `Proc`s, the result from `compute` will always be an array, even if you provide only one proc.

```ruby
irb(main):007:1* multi = ConcurrentRails::Multi.enqueue(
irb(main):008:1*   -> { 42 }
irb(main):009:0> )
=> #<ConcurrentRails::Multi:0x00007fbc403f0b98 @actions=[#<Proc:0x00007...

irb(main):010:0> multi.compute
=> [42]
```

Same as `Future`, one of the `Multi` tasks can fail. You can access the exception by calling `#errors`:

```ruby
irb(main):001:1*  multi = ConcurrentRails::Multi.enqueue(
irb(main):002:1*    -> { 42 },
irb(main):003:1*    -> { 2 / 0 }
irb(main):004:0>  )
=> #<ConcurrentRails::Multi:0x00007fb46d3ee3a0 @actions=[#<Proc:0x00007..

irb(main):005:0> multi.complete?
=> true

irb(main):006:0> multi.compute
=> [42, nil]

irb(main):007:0> multi.errors
=> [#<ZeroDivisionError: divided by 0>]
```

It is worth mention that a failed proc will return `nil`.

For more information on how Futures work and how Rails handle multithread check these links:

[Future documentation](https://github.com/ruby-concurrency/concurrent-ruby/blob/master/docs-source/future.md)

[Threading and code execution on rails](https://guides.rubyonrails.org/threading_and_code_execution.html)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'concurrent_rails'
```

And then execute:

```bash
$ bundle
```

Or install it yourself as:

```bash
$ gem install concurrent_rails
```

## Contributing

Pull-requests are always welcome

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
