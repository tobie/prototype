require 'rake'
require 'rake/packagetask'

PROTOTYPE_ROOT          = File.expand_path(File.dirname(__FILE__))
PROTOTYPE_SRC_DIR       = File.join(PROTOTYPE_ROOT, 'src')
PROTOTYPE_DIST_DIR      = File.join(PROTOTYPE_ROOT, 'dist')
PROTOTYPE_PKG_DIR       = File.join(PROTOTYPE_ROOT, 'pkg')
PROTOTYPE_TEST_DIR      = File.join(PROTOTYPE_ROOT, 'test')
PROTOTYPE_TEST_UNIT_DIR = File.join(PROTOTYPE_TEST_DIR, 'unit')
PROTOTYPE_TMP_DIR       = File.join(PROTOTYPE_TEST_UNIT_DIR, 'tmp')
PROTOTYPE_VERSION       = '1.6.0.3'

task :default => [:dist, :dist_helper, :package, :clean_package_source]

desc "Builds the distribution."
task :dist do
  $:.unshift File.join(PROTOTYPE_ROOT, 'lib')
  require 'protodoc'
  
  Dir.chdir(PROTOTYPE_SRC_DIR) do
    File.open(File.join(PROTOTYPE_DIST_DIR, 'prototype.js'), 'w+') do |dist|
      dist << Protodoc::Preprocessor.new('prototype.js')
    end
  end
end

desc "Builds the updating helper."
task :dist_helper do
  $:.unshift File.join(PROTOTYPE_ROOT, 'lib')
  require 'protodoc'
  
  Dir.chdir(File.join(PROTOTYPE_ROOT, 'ext', 'update_helper')) do
    File.open(File.join(PROTOTYPE_DIST_DIR, 'prototype_update_helper.js'), 'w+') do |dist|
      dist << Protodoc::Preprocessor.new('prototype_update_helper.js')
    end
  end
end

Rake::PackageTask.new('prototype', PROTOTYPE_VERSION) do |package|
  package.need_tar_gz = true
  package.package_dir = PROTOTYPE_PKG_DIR
  package.package_files.include(
    '[A-Z]*',
    'dist/prototype.js',
    'lib/**',
    'src/**',
    'test/**'
  )
end

task :clean_package_source do
  rm_rf File.join(PROTOTYPE_PKG_DIR, "prototype-#{PROTOTYPE_VERSION}")
end

desc %[Builds the Prototype distribution, then builds and runs the Prototype test suite.
The following options are available:
  
- BROWSERS: Lets you choose which browsers to run the tests on.
  For example, to run the test suite on Internet Explorer and Chrome:
      $ rake test BROWSERS=chrome,ie
  Currently supported browsers are:
      Internet Explorer       ie
      Firefox                 firefox
      Safari                  safari
      Opera                   opera
      Konqueror               konqueror
      Chrome                  chrome
  
- TESTS: Lets you specify which tests you want to build and run.
  For example, to run "array_test.js" and "hash_test.js":
      $ rake test TESTS=array,hash
  
- TESTCASES: Lets you specify which testcase to run.
  For example, to run test "testLastIndexOf" and "testIndexOf" of "array_test.js":
      $ rake test TESTS=array TESTCASES=testLastIndexOf,testIndexOf
]
task :test => ['test:build', 'test:run']
namespace :test do
  desc 'Runs the Prototype test suite. For available options, see "rake test".'
  task :run => [:require] do
    testcases        = ENV['TESTCASES']
    browsers_to_test = ENV['BROWSERS'] && ENV['BROWSERS'].split(',')
    tests_to_run     = ENV['TESTS'] && ENV['TESTS'].split(',')
    runner           = UnittestJS::WEBrickRunner::Runner.new(:test_dir => PROTOTYPE_TMP_DIR)

    Dir[File.join(PROTOTYPE_TMP_DIR, '*_test.html')].each do |file|
      file = File.basename(file)
      test = file.sub('_test.html', '')
      unless tests_to_run && !tests_to_run.include?(test)
        runner.add_test(file, testcases)
      end
    end
    
    UnittestJS::Browser::SUPPORTED.each do |browser|
      unless browsers_to_test && !browsers_to_test.include?(browser)
        runner.add_browser(browser.to_sym)
      end
    end
    
    trap('INT') { runner.teardown; exit }
    runner.run
  end
  
  desc 'Builds the Prototype distribution and test suite. For available options, see "rake test".'
  task :build => [:clean, :dist] do
    builder = UnittestJS::Builder::SuiteBuilder.new({
      :input_dir  => PROTOTYPE_TEST_UNIT_DIR,
      :assets_dir => PROTOTYPE_DIST_DIR
    })
    selected_tests = (ENV['TESTS'] || '').split(',')
    builder.collect(*selected_tests)
    builder.render
  end
  
  desc 'Empties the Prototype test suite directory.'
  task :clean => [:require] do
    UnittestJS::Builder.empty_dir!(PROTOTYPE_TMP_DIR)
  end
  
  task :require do
    lib = 'vendor/unittest_js/lib/unittest_js'
    unless File.exists?(lib)
      puts "\nYou'll need UnittestJS to run the tests. Just run:\n\n"
      puts "  $ git submodule init vendor/unittest_js"
      puts "  $ git submodule update vendor/unittest_js"
      puts "\nand you should be all set.\n\n"
    end
    require lib
    
    # UnittestJS currently relies on Prototype. We need to include a current 
    # version of Prototype, not the one built-in to the library. Hence the
    # monkey-pacthing below.
    class UnittestJS::Builder::TestBuilder
      def lib_files
        lib_assets = @options.output_unittest_assets_dir.name
        [
          to_script_tag("#{@options.output_assets_dir.name}/prototype.js"),
          to_script_tag("#{lib_assets}/unittest.js"),
          to_link_tag("#{lib_assets}/unittest.css")
        ].join("\n")
      end
    end
  end
end

task :test_units do
  puts '"rake test_units" is deprecated. Please use "rake test" instead.'
end

task :build_unit_tests do
  puts '"rake test_units" is deprecated. Please use "rake test:build" instead.'
end

task :clean_tmp do
  puts '"rake clean_tmp" is deprecated. Please use "rake test:clean" instead.'
end

namespace :caja do
  desc %[Builds the Prototype distribution, then builds and runs the cajoled Prototype test suite.
"rake caja:test" lets you specify the same set of options as "rake test" and the path to the Caja svn trunk.
    $ rake caja:test CAJA_SRC_PATH=/path/to/svn/trunk
If CAJA_SRC_PATH is not specified, "rake caja:test" defaults to its own copy of Caja which might be outdated.
]
  task :test => ['test:build', 'test:run']
  
  namespace :test do
    desc 'Runs the cajoled test suite. For available options, see "rake caja:test".'
    task :run => ['rake:test:run']
    
    desc 'Builds the Prototype distribution and cajoled test suite. For available options, see "rake caja:test".'
    task :build => [:require, 'rake:test:clean', :dist] do
      options = {
        :input_dir          => PROTOTYPE_TEST_UNIT_DIR,
        :assets_dir         => PROTOTYPE_DIST_DIR,
        :whitelist_dir      => File.join(PROTOTYPE_TEST_DIR, 'unit', 'caja_whitelists'),
        :html_attrib_schema => 'html_attrib.json'
      }
      
      options[:caja_dir] = ENV['CAJA_SRC_PATH'] if ENV['CAJA_SRC_PATH']
      builder = UnittestJS::CajaBuilder::SuiteBuilder.new(options)
      selected_tests = (ENV['TESTS'] || '').split(',')
      builder.collect(*selected_tests)
      builder.render
    end
  end
  task :require => ['rake:test:require'] do
    lib = 'vendor/caja_builder/lib/caja_builder'
    unless File.exists?(lib)
      puts "\nYou'll need UnittestJS to run the tests. Just run:\n\n"
      puts "  $ git submodule init vendor/caja_builder"
      puts "  $ git submodule update vendor/caja_builder"
      puts "\nand you should be all set.\n\n"
    end
    require lib
  end
  
  namespace :assets do
    desc %[Refreshes caja assets.
Useful when combined with "rake caja:test:run" as it avoids the lengthly process of cajoling the tests suite each time a change is made to Caja.
You'll need to provide it with the Caja source path you are currently working from.
    $ rake caja:test:assets:refresh CAJA_SRC_PATH=/path/to/svn/trunk
    $ rake caja:test:run CAJA_SRC_PATH=/path/to/svn/trunk
]
    task :refresh => ['caja:require'] do
      options = {
        :input_dir          => PROTOTYPE_TEST_UNIT_DIR,
        :assets_dir         => PROTOTYPE_DIST_DIR
      }
      raise "Please define CAJA_SRC_PATH." unless ENV['CAJA_SRC_PATH']
      options[:caja_dir] = ENV['CAJA_SRC_PATH']
      builder = UnittestJS::CajaBuilder::SuiteBuilder.new(options)
      builder.assets_handler.refresh
    end
  end
end