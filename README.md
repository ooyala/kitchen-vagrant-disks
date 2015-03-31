Creating additional disks in kitchen-vagrant/virtualbox
=======================================================

Notes on how to set up and use the hooks to create additional ephemeral disks
in kitchen-vagrant/virtualbox runs with test-kitchen.

Relevant files:
---------------
* .kitchen.yml
* scripts/write_vagrantfiles.rb
* Berksfile
* Rakefile
* Gemfile (Requires a minimum kitchen-vagrant version of 0.16.0)
* .gitignore

Other requirements:
-------------------
* ruby 2+ - kitchen-vagrant 0.16.0 is currently broken for older rubygems versions
* Enough local disk space to create your desired virtual disk

Set-up:
-------
1. Update your Gemfile to use kitchen-vagrant, version 0.16.0 or greater.  At
this time, it's still beta and not yet packaged in gem, so you'll need to pull
from github.com.
2. You'll need a recipe to format and mount the additional disk.  Add this
to the `.kitche.yml` at the beginning of the `run_list` for the suites that
need it.
3. In the `Rakefile`, the `:cleanup` namespace and the `integration:pre_cmds`
task are added to manage the setup and tear-down for the additional disk.
4. You'll want to add the following to the cookbook .gitignore:
```
.kitchen.local.yml
Vagrantfiles/
vagrant_disks/
```
5. The setup script looks for the attribute `add_vagrant_disk: true` in the
`.kitchen.yml` suites that need an additional disk.
6. By default, the added disk will be 10Gb.  For a different size, you can
add the attribute `vagrant_disk_size` to the suite (value should be an
integer, in Gb).

Execution:
----------
1. `bundle install`
2. `bundle exec berks install`
3. `bundle exec rake setup`
4. `bundle exec kitchen converge [target instance]`
5. Test your stuff
6. When you're ready to clean up, run `bundle exec rake cleanup`

Caveats:
--------
* If you need to update your `.kitchen.yml`, you will need to re-run
`bundle exec rake setup` each time, as the setup writes a `.kitchen.local.yml`
based on the `.kitchen.yml` file.
