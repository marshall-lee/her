require "bundler"
require "rake"
require "bundler/gem_tasks"
require "rspec/core/rake_task"

task :default => :spec

desc "Run all specs"
RSpec::Core::RakeTask.new(:spec) do |task|
  task.pattern = "spec/**/*_spec.rb"
end

task :benchmark do
  require 'her'
  require 'json'
  require 'benchmark'

  stubs = Faraday::Adapter::Test::Stubs.new do |stub|
    stub.get('/tamago') { |env| [200, {}, 'egg'] }
  end

  big_array = 10_000.times.map do |i|
    {
      id: i,
      email: "user#{i}@example.com",
      name: "Boris ##{i}"
    }
  end
  big_json = JSON.dump big_array

  Her::API.setup url: "https://api.example.com" do |faraday|
    faraday.use Her::Middleware::DefaultParseJSON
    faraday.adapter :test, stubs do |stub|
      stub.get('/users') { |env| [ 200, {}, big_json ]}
    end
  end

  class User
    include Her::Model
  end

  puts Benchmark.measure { User.all.to_a }
end
