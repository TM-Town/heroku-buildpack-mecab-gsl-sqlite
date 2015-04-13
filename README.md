Heroku buildpack: MeCab, GSL and SQLite3
======================

This is a buildpack that enables using the [mecab gem](https://rubygems.org/gems/mecab) and [Ruby/GSL gem](https://rubygems.org/gems/rb-gsl) and [sqlite3 gem](https://github.com/sparklemotion/sqlite3-ruby) on Heroku Cedar. This buildpack was forked from [heroku-buildpack-gsl-ruby](https://github.com/tomwolfe/heroku-buildpack-gsl-ruby). A big thank you to [jkatzer](https://github.com/jkatzer) who did 99.9% of the work to adapt the fork to get MeCab working on Heroku. Any mistakes are purely my own. Also thanks to [Tomasz](https://github.com/tomasz-buchta) for adding the sqlite3 gem.

To get MeCab and GSL working together on Heroku follow these steps:

1) Add the [mecab gem](https://rubygems.org/gems/mecab) and the [rb-gsl gem](https://rubygems.org/gems/rb-gsl) and [sqlite3 gem](https://github.com/sparklemotion/sqlite3-ruby) to your Gemfile and run bundle install  
`gem 'mecab', '0.996'`  
`gem 'rb-gsl', '~> 1.16.0.2'`  
`gem 'sqlite3'`  
`$ bundle install`  

2) Add the following config variables to your Heroku app  
`$ heroku config:set BUILDPACK_URL=https://github.com/TM-Town/heroku-buildpack-mecab-gsl-sqlite.git`  
`$ heroku config:set LD_LIBRARY_PATH=/app/vendor/gsl-1/lib:/app/vendor/mecab/lib`  
3) Add a myvendor folder to the root of your app with the `sqlite-autoconf-3080803.tar.gz` file inside

4) Add the folder heroku_buildpack_scripts and a symlink folder hbs linking to that folder. Inside the heroku_buildpack_scripts show be a bash script `dothis.bash`

```
#!/bin/bash

# hbs/dothis.bash

# I use this script to help me enhance the ability of
# heroku-buildpack

# heroku-buildpack is software used by heroku to help me deploy applications.

rails_root=`pwd`

echo rails_root is
echo $rails_root
cd ${rails_root}/myvendor/

# I should install sqlite3 software locally under Rails.root

mkdir -p ${rails_root}/myvendor/sqlite3
tar zxf  ${rails_root}/myvendor/sqlite-autoconf-3080803.tar.gz
cd sqlite-autoconf-3080803/
./configure --prefix=${rails_root}/myvendor/sqlite3 > /dev/null
make -s
make install -s

cd ${rails_root}
# I should be able to gem install sqlite3 now
gem install sqlite3 -- --with-sqlite3-dir=${rails_root}/myvendor/sqlite3
gem list    sqlite3

exit

```

5) Push your app to Heroku  
`$ git push heroku master`  


Heroku buildpack: Ruby
======================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for Ruby, Rack, and Rails apps. It uses [Bundler](http://gembundler.com) for dependency management.

Usage
-----

### Ruby

Example Usage:

    $ ls
    Gemfile Gemfile.lock

    $ heroku create --stack cedar --buildpack https://github.com/heroku/heroku-buildpack-ruby.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Ruby app detected
    -----> Installing dependencies using Bundler version 1.1.rc
           Running: bundle install --without development:test --path vendor/bundle --deployment
           Fetching gem metadata from http://rubygems.org/..
           Installing rack (1.3.5)
           Using bundler (1.1.rc)
           Your bundle is complete! It was installed into ./vendor/bundle
           Cleaning up the bundler cache.
    -----> Discovering process types
           Procfile declares types -> (none)
           Default types for Ruby  -> console, rake

The buildpack will detect your app as Ruby if it has a `Gemfile` and `Gemfile.lock` files in the root directory. It will then proceed to run `bundle install` after setting up the appropriate environment for [ruby](http://ruby-lang.org) and [Bundler](http://gembundler.com).

#### Run the Tests

The tests on this buildpack are written in Rspec to allow the use of
`focused: true`. Parallelization of testing is provided by
https://github.com/grosser/parallel_tests this lib spins up an arbitrary
number of processes and running a different test file in each process,
it does not parallelize tests within a test file. To run the tests: clone the repo, then `bundle install` then clone the test fixtures by running:

```sh
$ hatchet install
```

Now run the tests:

```sh
$ bundle exec parallel_rspec -n 6 spec/
```

If you don't want to run them in parallel you can still:

```sh
$ bundle exec rake spec
```

Now go take a nap or something for a really long time.

#### Bundler

For non-windows `Gemfile.lock` files, the `--deployment` flag will be used. In the case of windows, the Gemfile.lock will be deleted and Bundler will do a full resolve so native gems are handled properly. The `vendor/bundle` directory is cached between builds to allow for faster `bundle install` times. `bundle clean` is used to ensure no stale gems are stored between builds.

### Rails 2

Example Usage:

    $ ls
    app  config  db  doc  Gemfile  Gemfile.lock  lib  log  public  Rakefile  README  script  test  tmp  vendor

    $ ls config/environment.rb
    config/environment.rb

    $ heroku create --stack cedar --buildpack https://github.com/heroku/heroku-buildpack-ruby.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Ruby/Rails app detected
    -----> Installing dependencies using Bundler version 1.1.rc
    ...
    -----> Writing config/database.yml to read from DATABASE_URL
    -----> Rails plugin injection
           Injecting rails_log_stdout
    -----> Discovering process types
           Procfile declares types      -> (none)
           Default types for Ruby/Rails -> console, rake, web, worker

The buildpack will detect your app as a Rails 2 app if it has a `environment.rb` file in the `config`  directory.

#### Rails Log STDOUT
  A [rails_log_stdout](http://github.com/ddollar/rails_log_stdout) is installed by default so Rails' logger will log to STDOUT and picked up by Heroku's [logplex](http://github.com/heroku/logplex).

#### Auto Injecting Plugins

Any vendored plugin can be stopped from being installed by creating the directory it's installed to in the slug. For instance, to prevent rails_log_stdout plugin from being injected, add `vendor/plugins/rails_log_stdout/.gitkeep` to your git repo.

### Rails 3

Example Usage:

    $ ls
    app  config  config.ru  db  doc  Gemfile  Gemfile.lock  lib  log  Procfile  public  Rakefile  README  script  tmp  vendor

    $ ls config/application.rb
    config/application.rb

    $ heroku create --stack cedar --buildpack https://github.com/heroku/heroku-buildpack-ruby.git

    $ git push heroku master
    -----> Heroku receiving push
    -----> Ruby/Rails app detected
    -----> Installing dependencies using Bundler version 1.1.rc
           Running: bundle install --without development:test --path vendor/bundle --deployment
           ...
    -----> Writing config/database.yml to read from DATABASE_URL
    -----> Preparing app for Rails asset pipeline
           Running: rake assets:precompile
    -----> Rails plugin injection
           Injecting rails_log_stdout
           Injecting rails3_serve_static_assets
    -----> Discovering process types
           Procfile declares types      -> web
           Default types for Ruby/Rails -> console, rake, worker

The buildpack will detect your apps as a Rails 3 app if it has an `application.rb` file in the `config` directory.

#### Assets

To enable static assets being served on the dyno, [rails3_serve_static_assets](http://github.com/pedro/rails3_serve_static_assets) is installed by default. If the [execjs gem](http://github.com/sstephenson/execjs) is detected then [node.js](http://github.com/joyent/node) will be vendored. The `assets:precompile` rake task will get run if no `public/manifest.yml` is detected.  See [this article](http://devcenter.heroku.com/articles/rails31_heroku_cedar) on how rails 3.1 works on cedar.

Hacking
-------

To use this buildpack, fork it on Github.  Push up changes to your fork, then create a test app with `--buildpack <your-github-url>` and push to it.

To change the vendored binaries for Bundler, [Node.js](http://github.com/joyent/node), and rails plugins, use the rake tasks provided by the `Rakefile`. You'll need an S3-enabled AWS account and a bucket to store your binaries in as well as the [vulcan](http://github.com/heroku/vulcan) gem to build the binaries on heroku.

For example, you can change the vendored version of Bundler to 1.1.rc.

First you'll need to build a Heroku-compatible version of Node.js:

    $ export AWS_ID=xxx AWS_SECRET=yyy S3_BUCKET=zzz
    $ s3 create $S3_BUCKET
    $ rake gem:install[bundler,1.1.rc]

Open `lib/language_pack/ruby.rb` in your editor, and change the following line:

    BUNDLER_VERSION = "1.1.rc"

Open `lib/language_pack/base.rb` in your editor, and change the following line:

    VENDOR_URL = "https://s3.amazonaws.com/zzz"

Commit and push the changes to your buildpack to your Github fork, then push your sample app to Heroku to test.  You should see:

    -----> Installing dependencies using Bundler version 1.1.rc

NOTE: You'll need to vendor the plugins, node, Bundler, and libyaml by running the rake tasks for the buildpack to work properly.

Flow
----

Here's the basic flow of how the buildpack works:

Ruby (Gemfile and Gemfile.lock is detected)

* runs Bundler
* installs binaries
  * installs node if the gem execjs is detected
* runs `rake assets:precompile` if the rake task is detected

Rack (config.ru is detected)

* everything from Ruby
* sets RACK_ENV=production

Rails 2 (config/environment.rb is detected)

* everything from Rack
* sets RAILS_ENV=production
* install rails 2 plugins
  * [rails_log_stdout](http://github.com/ddollar/rails_log_stdout)

Rails 3 (config/application.rb is detected)

* everything from Rails 2
* install rails 3 plugins
  * [rails3_server_static_assets](https://github.com/pedro/rails3_serve_static_assets)

**MeCab License Info**

MeCab is copyrighted free software by Taku Kudo taku@chasen.org and
Nippon Telegraph and Telephone Corporation, and is released under
any of the GPL (see the file GPL), the LGPL (see the file LGPL), or the
BSD License (see the file BSD).

Copyright (c) 2001-2008, Taku Kudo
Copyright (c) 2004-2008, Nippon Telegraph and Telephone Corporation
All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are
permitted provided that the following conditions are met:

Redistributions of source code must retain the above
copyright notice, this list of conditions and the
following disclaimer.

Redistributions in binary form must reproduce the above
copyright notice, this list of conditions and the
following disclaimer in the documentation and/or other
materials provided with the distribution.

Neither the name of the Nippon Telegraph and Telegraph Corporation
nor the names of its contributors may be used to endorse or
promote products derived from this software without specific
prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED
WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR
ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR
TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.