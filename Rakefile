require 'rake'
require 'rake/clean'
require 'rake/rdoctask'
require 'rake/gempackagetask'
require 'fileutils'
include FileUtils

# copied from EventMachine.
MAKE = ENV['MAKE'] || if RUBY_PLATFORM =~ /mswin/ # mingw uses make.
  'nmake'
else
  'make'
end

# Default Rake task is compile
task :default => :compile

# RDoc
Rake::RDocTask.new(:rdoc) do |task|
  task.rdoc_dir = 'doc'
  task.title    = 'EventMachine::HttpRequest'
  task.options = %w(--title HttpRequest --main README --line-numbers)
  task.rdoc_files.include(['lib/**/*.rb'])
  task.rdoc_files.include(['README', 'LICENSE'])
end

# Rebuild parser Ragel
task :ragel do
  Dir.chdir "ext/http11_client" do
    target = "http11_parser.c"
    File.unlink target if File.exist? target
    sh "ragel http11_parser.rl | rlgen-cd -G2 -o #{target}"
    raise "Failed to build C source" unless File.exist? target
  end
end

task :spec do
	sh 'spec spec/*_spec.rb'
end

def make(makedir)
  Dir.chdir(makedir) { sh MAKE }
end

def extconf(dir)
  Dir.chdir(dir) { ruby "extconf.rb" }
end

if RUBY_PLATFORM =~ /java/
  def java_classpath_arg
    # A myriad of ways to discover the JRuby classpath
    classpath = begin
      require 'java'
      # Already running in a JRuby JVM
      Java::java.lang.System.getProperty('java.class.path')
    rescue LoadError
      ENV['JRUBY_PARENT_CLASSPATH'] || ENV['JRUBY_HOME'] && FileList["#{ENV['JRUBY_HOME']}/lib/*.jar"].join(File::PATH_SEPARATOR)
    end
    classpath ? "-cp #{classpath}" : ""
  end
end

def define_extension_tasks
  extension = "http11_client"

  case RUBY_PLATFORM
  when /java/
    # Avoid JRuby in-process launching problem
    begin
      require 'jruby'
      JRuby.runtime.instance_config.run_ruby_in_process = false 
    rescue LoadError
    end

    filename = "lib/#{extension}.jar"
    file filename do
      build_dir = "ext/http11_java/classes"
      mkdir_p build_dir
      sources = FileList['ext/http11_java/**/*.java'].join(' ')
      sh "javac -target 1.4 -source 1.4 -d #{build_dir} #{java_classpath_arg} #{sources}"
      sh "jar cf #{filename} -C #{build_dir} ."
      #mv "ext/#{filename}", "lib/"
    end
    task extension.intern => [filename]

  else
    ext = "ext/#{extension}"
    ext_so = "#{ext}/#{extension}.#{Config::CONFIG['DLEXT']}"
    ext_files = FileList[
      "#{ext}/*.c",
      "#{ext}/*.h",
      "#{ext}/extconf.rb",
      "#{ext}/Makefile",
      "lib"
    ] 

    task "lib" do
      directory "lib"
    end

    mf = (extension + '_makefile').to_sym

    task mf do |t|
      extconf "#{ext}"
    end

    desc "Builds just the #{extension} extension"

    task extension.to_sym => [mf] do
      make "#{ext}"
      cp ext_so, "lib"
    end

  end
end

#setup_extension("buffer", "em_buffer")
#setup_extension("http11_client", "http11_client")

#task :compile => [:em_buffer, :http11_client]
define_extension_tasks

desc "compile extensions"
task :compile => [:http11_client]

CLEAN.include ['build/*', '**/*.o', '**/*.so', '**/*.a', '**/*.log', 'pkg']
CLEAN.include ['ext/buffer/Makefile', 'lib/em_buffer.*', 'lib/http11_client.*']
CLEAN.include ['ext/http11_java/classes/**/*','lib/http11_client.jar']

begin
  require 'jeweler'
  Jeweler::Tasks.new do |gemspec|
    gemspec.name = "em-http-request"
    gemspec.summary = "EventMachine based, async HTTP Request interface"
    gemspec.description = gemspec.summary
    gemspec.email = "ilya@igvita.com"
    gemspec.homepage = "http://github.com/igrigorik/em-http-request"
    gemspec.authors = ["Ilya Grigorik"]
    gemspec.required_ruby_version = ">= 1.8.6"
    if RUBY_PLATFORM !~ /java/
      gemspec.extensions = ["ext/buffer/extconf.rb" , "ext/http11_client/extconf.rb"]
    end
    gemspec.add_dependency('eventmachine', '>= 0.12.9')
    gemspec.add_dependency('addressable', '>= 2.0.0')
    gemspec.rubyforge_project = "em-http-request"
    gemspec.files = FileList[`git ls-files`.split]
    gemspec.files << "lib/http11_client.jar" if RUBY_PLATFORM =~ /java/
  end
  
  Jeweler::GemcutterTasks.new
rescue LoadError
  puts "Jeweler not available. Install it with: sudo gem install technicalpickles-jeweler -s http://gems.github.com"
end
