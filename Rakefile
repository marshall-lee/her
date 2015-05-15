require "bundler"
require "rake"
require "bundler/gem_tasks"
require "rspec/core/rake_task"

task :default => :spec

desc "Run all specs"
RSpec::Core::RakeTask.new(:spec) do |task|
  task.pattern = "spec/**/*_spec.rb"
end

task :benchmark_pool do
  require 'typhoeus/adapters/faraday'
  require 'her'
  require 'benchmark'

  class Suite
    def initialize(url)
      @url = url
    end

    attr_reader :url
    attr_accessor :use_pool

    def warming(*)
    end

    def setup!
      Object.send :remove_const, :User if Object.const_defined? :User
      if use_pool
        setup_pool
      else
        setup_single
      end
    end

    def running(*)
    end

    def warmup_stats(*)
    end

    def add_report(*)
      puts 'add_rep'
    end

    def fetch_one_user
      ::User.find('123')
    end

    def fetch_users
      ::User.all.to_a
    end

    def profile!
      result = RubyProf.profile do
        1000.times { fetch_users }
      end
      printer = RubyProf::FlatPrinter.new(result)
      printer.print(STDOUT)
    end

    def setup_pool
      Her::API.setup_pool 10, url: url do |c|
        # Request
        c.use Faraday::Request::UrlEncoded

        # Response
        c.use Her::Middleware::DefaultParseJSON

        # Adapter
        # c.use Faraday::Adapter::Patron
        # c.use Faraday::Adapter::NetHttpPersistent
        # c.use Faraday::Adapter::HTTPClient
        c.use Faraday::Adapter::Typhoeus
      end
      new_class = Class.new
      Object.const_set :User, new_class
      new_class.send :include, Her::Model
      puts "setup pool"
    end

    def setup_single
      Her::API.setup url: url do |c|
        # Request
        c.use Faraday::Request::UrlEncoded

        # Response
        c.use Her::Middleware::DefaultParseJSON

        # Adapter
        # c.use Faraday::Adapter::Patron
        # c.use Faraday::Adapter::NetHttpPersistent
        # c.use Faraday::Adapter::HTTPClient
        c.use Faraday::Adapter::Typhoeus
      end
      new_class = Class.new
      Object.const_set :User, new_class
      new_class.send :include, Her::Model
      puts "setup single"
    end
  end

  suite = Suite.new 'http://localhost:9292'

  suite.use_pool = false
  suite.setup!
  puts 'measure single threaded'
  puts Benchmark.realtime { 1000.times { suite.fetch_one_user } }
  puts 'measure multi threaded'
  puts Benchmark.realtime { 100.times.map { Thread.new { 10.times { suite.fetch_one_user } } }.each(&:join) }

  suite.use_pool = true
  suite.setup!
  puts 'measure single threaded'
  puts Benchmark.realtime { 1000.times { suite.fetch_one_user } }
  puts 'measure multi threaded'
  puts Benchmark.realtime { 100.times.map { Thread.new { 10.times { suite.fetch_one_user } } }.each(&:join) }

  puts `lsof -a -p #{Process.pid} | wc -l`
end
