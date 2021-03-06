require "rubygems"
require "rake/gempackagetask"
require "rake/testtask"
require "zlib"
require "rake/clean"

$:.unshift(File.dirname(__FILE__) + '/lib')
require 'htmlentities/version'

CLEAN.include("doc")
DOCTYPES = %w[html4 xhtml1]
DATA_FILES = DOCTYPES.map{ |d| "lib/htmlentities/#{d}.rb" }
SOURCES = FileList["lib/**/*.rb"] - DATA_FILES

def interpreters
  path = ENV["PATH"].split(":")
  Dir["{" + path * "," + "}/*ruby*"].
  select{ |path| File.basename(path) =~ /^j?ruby([\d\.]*)$/ }.
  inject({}){ |hash, path|
    version = `"#{path}" --version`
    hash[version] = path if version =~ /ruby/i
    hash
  }
end

task :default => :test

spec = Gem::Specification.new do |s|
  s.name   = "htmlentities"
  s.version = HTMLEntities::VERSION::STRING
  s.author = "Paul Battley"
  s.email = "pbattley@gmail.com"
  s.summary = "A module for encoding and decoding (X)HTML entities."
  s.files = FileList['{lib,test,perf}/**/*.rb', 'setup.rb']
  s.require_path = "lib"
  s.test_file = 'test/test_all.rb'
  s.has_rdoc = true
  s.extra_rdoc_files = %w[README.txt History.txt COPYING.txt]
  s.homepage = 'http://htmlentities.rubyforge.org/'
end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.need_tar_gz = true
end

Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = FileList['test/*_test.rb']
  t.verbose = true
end

namespace :test do
  desc "Run tests with all available Ruby interpreters"
  task :versions do |t|
    interpreters.each do |version, path|
      puts version
      system path, "test/test_all.rb"
    end
  end
end

DOCTYPES.each do |name|
  file "lib/htmlentities/#{name}.rb" => %w[util/build_entities.rb] do |f|
    rm_f f.name
    sh %{ruby util/build_entities.rb #{name} > #{f.name}}
  end
end

file "doc" => SOURCES do |f|
  sh %{rdoc #{SOURCES.join(' ')}}
end

task :entities => DATA_FILES

desc "Run benchmark with all available Ruby interpreters"
task :benchmark do |t|
  interpreters.each do |version, path|
    puts version
    system path, "perf/benchmark.rb"
  end
end

desc "Use profiler to analyse encoding and decoding"
task :profile do |t|
  system "ruby", "perf/profile.rb"
end
