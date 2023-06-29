require 'bundler/gem_tasks'
require 'rake/testtask'

Rake::TestTask.new(:test) do |test|
  test.ruby_opts << '-r./test/simplecov_start.rb' if RUBY_VERSION >= '1.9'
  test.pattern = 'test/**/test_*.rb'
  test.verbose = true
end

task :default => :test

require 'rdoc/task'
Rake::RDocTask.new do |rdoc|
  version = HTTP::Cookie::VERSION

  rdoc.rdoc_dir = 'rdoc'
  rdoc.title = "http-cookie #{version}"
  rdoc.rdoc_files.include('lib/**/*.rb')
  rdoc.rdoc_files.include(Bundler::GemHelper.gemspec.extra_rdoc_files)
end

task :thread_safety_check, [:thread_count] do |_, args|
  thread_count = Integer(args[:thread_count] || "1000", 10)
  $LOAD_PATH.unshift(File.join(__dir__, 'lib'))

  require 'http/cookie' or raise "already required"

  # see https://github.com/sparklemotion/http-cookie/issues/27
  puts "checking for thread safety bug with #{thread_count} threads on #{RUBY_DESCRIPTION}"

  results = thread_count.times.map do
    Thread.new do
      begin
        HTTP::CookieJar::HashStore.new
        nil
      rescue => e
        e
      end
    end
  end.map(&:value)

  exceptions = results.reject(&:nil?)
  if exceptions.any?
    classes = exceptions.map { |e| e.class.name }.uniq
    warn "#{exceptions.size} of #{thread_count} threads saw exceptions: #{classes.inspect}"
    raise exceptions.first
  else
    puts "no issues detected"
  end
end
