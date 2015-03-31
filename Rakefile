# Copyright 2015 Ooyala, Inc. All rights reserved.
#
# This file is licensed under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance with the
# License. You may obtain a copy of the License at
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.  See the License for the specific language governing
# permissions and limitations under the License.

require 'rspec/core/rake_task'
require 'rubocop/rake_task'
require 'foodcritic'
require 'mixlib/shellout'
require 'kitchen/rake_tasks'

# Style tests. Rubocop and Foodcritic
namespace :style do
  desc 'Run Ruby style checks'
  RuboCop::RakeTask.new(:ruby)

  desc 'Run Chef style checks'
  FoodCritic::Rake::LintTask.new(:chef) do |t|
    t.options = {
      fail_tags: ['any'],
      tags: ['~FC005']
    }
  end
end

desc 'Run all style checks'
task style: %w(style:chef style:ruby)

# Rspec and ChefSpec
desc 'Run ChefSpec examples'
RSpec::Core::RakeTask.new(:spec) do |r|
  r.pattern = 'spec/*_spec.rb'
end

# Integration tests. Kitchen.ci
namespace :integration do
  desc 'Run test-kitchen pre-processing scripts'
  task :pre_cmds do
    cmd = Mixlib::ShellOut.new('./scripts/write_vagrantfile.rb')
    cmd.run_command
  end
end

# Clean up
namespace :cleanup do
  desc 'Destroy test-kitchen instances'
  task :kitchen_destroy do
    destroy = Kitchen::RakeTasks.new do |obj|
      def obj.destroy
        config.instances.each(&:destroy)
      end
    end
    destroy.destroy
  end

  desc 'Remove vagrant disks'
  task :rm_vdi do
    ::FileUtils.rm_rf('./vagrant_disks')
  end

  desc 'Remove Vagrantfiles/ dir'
  task :rm_vagrantfiles do
    ::FileUtils.rm_rf('./Vagrantfiles')
  end

  desc 'Remove .kitchen.local.yml'
  task :rm_kitchen_local do
    ::File.unlink('.kitchen.local.yml') if ::File.exist?('.kitchen.local.yml')
  end
end

desc 'Run full integration'
task integration: %w(integration:pre_cmds integration:vagrant)

desc 'Generate the setup'
task setup: %w(integration:pre_cmds)

desc 'Clean up generated files'
task cleanup: %w(cleanup:kitchen_destroy cleanup:rm_vdi
                 cleanup:rm_kitchen_local cleanup:rm_vagrantfiles)

desc 'Alias for integration'
task kitchen: %w(integration:pre_cmds kitchen:all)

# Default
task default: %w(style spec integration)
