Slop
====

Slop is a simple option parser with an easy to remember syntax and friendly API.
API Documentation is available [here](http://leejarvis.github.com/rdoc/slop/).

[![Build Status](https://travis-ci.org/leejarvis/slop.png?branch=master)](http://travis-ci.org/leejarvis/slop)

Usage
-----

```ruby
opts = Slop.parse do
  banner 'Usage: foo.rb [options]'

  on 'name=', 'Your name'
  on 'p', 'password', 'An optional password', optional_argument: true
  on 'v', 'verbose', 'Enable verbose mode'
end

# if ARGV is `--name Lee -v`
opts.verbose?  #=> true
opts.password? #=> false
opts[:name]    #=> 'lee'
opts.to_hash   #=> {:name=>"Lee", :password=>nil, :verbose=>true}
```

Advanced Usage
--------------

```ruby
Slop.parse do
  on :v, :verbose, 'Enable verbose mode'

  cmd :add, ignore_case: true do
    on :f, :force, 'Force add'
    on :d, :dir=,  'Output directory'

    process do |cmd, opts|
      puts "Adding #{opts.first} to #{cmd[:dir]} ..."
    end
  end

  cmd :rm do
    on :f, :force, 'Force rm'

    process do |cmd, opts|
      unless cmd.force?
        abort "--force must be present"
      end
    end
  end

end
```

Installation
------------

    gem install slop

Printing Help
-------------

Slop attempts to build a good looking help string to print to your users. You
can see this by calling `opts.help` or simply `puts opts`.

Configuration Options
---------------------

Key                 | Description                                                 | Default
--------------------|----------------------------------------------------------------------
`strict`            | Raise `InvalidOptionError` for unknown options              | true
`help`              | Automatically add the `--help` option                       | true
`ignore_case`       | `-A` will be processed the same as `-a`                     | false
`multiple_switches` | When disabled, parses `-abc` as `-a` with the argument `bc` | true

Lists
-----

```ruby
opts = Slop.parse do
  on :list=, as: Array
end
# ruby run.rb --list one,two
opts[:list] #=> ["one", "two"]
# ruby run.rb --list one,two --list three
opts[:list] #=> ["one", "two", "three"]
```

You can also specify a delimiter and limit.

```ruby
opts = Slop.parse do
  on :list=, as: Array, delimiter: ':', limit: 2
end
# ruby run.rb --list one:two:three
opts[:list] #=> ["one", "two:three"]
```

Ranges
------

```ruby
opts = Slop.parse do
  on :range=, as: Range
end
# ruby run.rb --range 1..10
opts[:range] #=> 1..10
# ruby run.rb --range 1...10
opts[:range] #=> 1...10
# ruby run.rb --range 1-10
opts[:range] #=> 1..10
# ruby run.rb --range 1,10
opts[:range] #=> 1..10
```

Autocreate
----------

Slop has an 'autocreate' feature. This feature is intended to create
options on the fly, without having to specify them yourself. In some case,
using this code could be all you need in your application:

```ruby
# ruby run.rb --foo bar --baz --name lee
opts = Slop.parse(autocreate: true)
opts.to_hash #=> {:foo=>"bar", :baz=>true, :name=>"lee"}
opts.fetch_option(:name).expects_argument? #=> true
```

Commands
--------

Slop supports git style sub-commands, like so:

```ruby
opts = Slop.parse do
  on '-v', 'Print the version' do
    puts "Version 1.0"
  end

  command 'add' do
    on :v, :verbose, 'Enable verbose mode'
    on :name, 'Your name'

    run do |opts, args|
      puts "You ran 'add' with options #{opts.to_hash} and args: #{args.inspect}"
    end
  end
end

# ruby run.rb -v
#=> Version 1.0
# ruby run.rb add -v foo --name Lee
#=> You ran 'add' with options {:verbose=>true,:name=>"Lee"} and args ["foo"]
opts.to_hash(true) # Pass true to tell Slop to merge sub-command option values.
# => { :v => nil, :add => { :v => true, :name => "Lee" } }
```

Woah woah, why you hating on OptionParser?
------------------------------------------

I'm not, honestly! I love OptionParser. I really do, it's a fantastic library.
So why did I build Slop? Well, I find myself using OptionParser to simply
gather a bunch of key/value options, usually you would do something like this:

```ruby
require 'optparse'

things = {}

opt = OptionParser.new do |opt|
  opt.on('-n', '--name NAME', 'Your name') do |name|
    things[:name] = name
  end

  opt.on('-a', '--age AGE', 'Your age') do |age|
    things[:age] = age.to_i
  end

  # you get the point
end

opt.parse
things #=> { :name => 'lee', :age => 105 }
```

Which is all great and stuff, but it can lead to some repetition. The same
thing in Slop:

```ruby
require 'slop'

opts = Slop.parse do
  on :n, :name=, 'Your name'
  on :a, :age=, 'Your age', as: Integer
end

opts.to_hash #=> { :name => 'lee', :age => 105 }
```
