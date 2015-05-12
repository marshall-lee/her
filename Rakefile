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
  require 'her'
  require 'benchmark'

  url = 'http://localhost:9292'
  # url = 'http://apifake.herokuapp.com'

  fetch_users = -> {
    10.times.map do
      Thread.new do
        10.times do
          User.all.to_a
        end
      end
    end.each(&:join)
  }

  Her::API.setup_pool 5, url: url do |c|
    # Request
    c.use Faraday::Request::UrlEncoded

    # Response
    c.use Her::Middleware::DefaultParseJSON

    # Adapter
    c.use Faraday::Adapter::Patron
  end
  class User
    include Her::Model
  end
  fetch_users.call
  puts Benchmark.measure(&fetch_users)

  Object.send :remove_const, :User

  Her::API.setup url: url do |c|
    # Request
    c.use Faraday::Request::UrlEncoded

    # Response
    c.use Her::Middleware::DefaultParseJSON

    # Adapter
    c.use Faraday::Adapter::Patron
  end
  class User
    include Her::Model
  end
  puts Benchmark.measure(&fetch_users)

end
