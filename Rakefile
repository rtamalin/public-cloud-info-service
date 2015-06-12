require 'rspec/core/rake_task'
require 'bundler'

task :default => ['specs']

RSpec::Core::RakeTask.new :specs do |task|
  task.pattern = Dir['spec/**/*_spec.rb']
end

namespace :gems do
  task :unlock, [:env] do |task, args|
    if args.env
      rm "Gemfile.#{args.env}.lock"
    else
      rm Dir.glob("Gemfile.*.lock")
    end
  end
  
  task :lock, [:env] => [:unlock] do |task, args|
    if args.env
      system "bundle install --local --gemfile=Gemfile.#{args.env}"
    else
      Dir.glob("Gemfile.*").each do |gemfile|
        system "bundle install --local --gemfile=#{gemfile}"
      end
    end
  end

  task :rpmlist, [:ruby_version, :env] do |task, args|
    ruby_version = args.ruby_version || RUBY_VERSION.split(".")[0,2].join(".")
    if args.env
      filenames = ["Gemfile.#{args.env}.lock"]
    else
      filenames = Dir.glob("Gemfile.*.lock")
    end
    filenames.each do |filename|
      puts "#{filename}:"
      gemspecs(filename).each do |gemspec|
        puts "\truby#{ruby_version}-rubygem-#{gemspec.name}"
      end
    end
  end
  
  namespace :rpmspec do
    task :requires do |task|
      puts "BuildRequires:  ruby-macros >= 5"
      puts "Requires:  %{ruby}"
      gemspecs("Gemfile.production.lock").each do |gemspec|
        puts "Requires:  %{rubygem #{gemspec}}"
      end
    end
  end
end

def gemspecs(gemfile)
  ENV['BUNDLE_GEMFILE'] = gemfile
  lockfile = Bundler::LockfileParser.new(Bundler.read_file(gemfile))
  lockfile.specs.collect(&:name).sort
end
