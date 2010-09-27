require 'rubygems'
require 'rake'
require 'cucumber/rake/task'
require 'date'

TEST_PROJECT = 'test_project'
SUSPENDERS_GEM_VERSION = '0.0.4'

#############################################################################
#
# Testing functions
#
#############################################################################

Cucumber::Rake::Task.new

namespace :test do
  desc "A full suspenders app's test suite"
  task :full => ['test_project:generate', 'cucumber', 'test_project:destroy']
end

namespace :test_project do
  desc 'Suspend a new project. Pass REPO=... to change the Suspenders repo.'
  task :generate do
    FileUtils.rm_rf(TEST_PROJECT)
    sh './bin/suspenders', 'create', TEST_PROJECT, ENV['REPO'].to_s
  end

  desc 'Remove a suspended project'
  task :destroy do
    FileUtils.cd TEST_PROJECT
    sh "rake db:drop RAILS_ENV=development"
    sh "rake db:drop RAILS_ENV=test"
    FileUtils.cd '..'
    FileUtils.rm_rf TEST_PROJECT
  end
end

desc 'Run the test suite'
task :default => ['test:full']

#############################################################################
#
# Helper functions
#
#############################################################################

def name
  @name ||= Dir['*.gemspec'].first.split('.').first
end

def version
  SUSPENDERS_GEM_VERSION
end

def date
  Date.today.to_s
end

def gemspec_file
  "#{name}.gemspec"
end

def gem_file
  "#{name}-#{version}.gem"
end

def replace_header(head, header_name)
  head.sub!(/(\.#{header_name}\s*= ').*'/) { "#{$1}#{send(header_name)}'"}
end

#############################################################################
#
# Packaging tasks
#
#############################################################################

task :release => :build do
  unless `git branch` =~ /^\* master$/
    puts "You must be on the master branch to release!"
    exit!
  end
  sh "git commit --allow-empty -a -m 'Release #{version}'"
  sh "git tag v#{version}"
  sh "git push origin master"
  sh "git push --tags"
  sh "gem push pkg/#{name}-#{version}.gem"
end

task :build => :gemspec do
  sh "mkdir -p pkg"
  sh "gem build #{gemspec_file}"
  sh "mv #{gem_file} pkg"
end

task :gemspec do
  # read spec file and split out manifest section
  spec = File.read(gemspec_file)
  head, manifest, tail = spec.split("  # = MANIFEST =\n")

  # replace name version and date
  replace_header(head, :name)
  replace_header(head, :version)
  replace_header(head, :date)

  # determine file list from git ls-files
  files = `git ls-files`.
    split("\n").
    sort.
    reject { |file| file =~ /^\./ }.
    reject { |file| file =~ /^(rdoc|pkg)/ }.
    map { |file| "    #{file}" }.
    join("\n")

  # piece file back together and write
  manifest = "  s.files = %w[\n#{files}\n  ]\n"
  spec = [head, manifest, tail].join("  # = MANIFEST =\n")
  File.open(gemspec_file, 'w') { |io| io.write(spec) }
  puts "Updated #{gemspec_file}"
end
