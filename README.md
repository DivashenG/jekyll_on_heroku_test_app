# Jekyll on Heroku Test App
POC to test deploying a Jekyll App to Heroku

# Steps
___
## 1. Make a Jekyll blog
```
gem install jekyll
jekyll new jekyll_on_heroku_test_app
cd jekyll_on_heroku_test_app
jekyll serve
```
Now if you open http://localhost:4000 in your browser

You may get an error on `bundle install` -> `Failed to install gems via Bundler.` 
If the failure mentions `x86_64-linux` then run:
```
bundle lock --add-platform x86_64-linux
```
and commit the Gemfile.lock
___
## 2. Turn the blog into a Rack app
Let’s get a Gemfile set up with all the stuff you’ll need going forwards. Create a Gemfile file that includes:
```
source 'https://rubygems.org'
ruby '2.0.0'
gem 'bundler'
gem 'jekyll', '~>2.5.3'
gem 'rack-contrib'
gem 'kramdown'
gem 'puma'
gem 'rake'
Run bundle install from the command line. 
```

Next create a config.ru file. This tells Rack how to run your app.

```
require 'rack/contrib/try_static'
require 'rack/contrib/not_found'

use Rack::TryStatic,
:root => "_site",
:urls => %w[/],
:try  => ['index.html', '/index.html']

run Rack::NotFound.new('_site/404.html')
```
Run jekyll serve to make sure this page is generated and placed in your _site folder.

You need to put some extra stuff in the _config.yml file, particularly vendor so that Jekyll doesn’t get confused when it generates your site and looks in the wrong place for stuff. Add this line to the bottom:

```
exclude: ['config.ru', 'Gemfile', 'Gemfile.lock', 'vendor', 'Procfile']
```
Make a Rakefile and add a task that Heroku will use when building the site:

```
namespace :assets do
    task :precompile do
        puts `bundle exec jekyll build`
    end
end
```
Test your blog works as a Rack app: 
from the command line run the command rackup and open the result in your browser, in my case localhost:9292. 
You should see the same pages as when you did jekyll serve before. 
Congrats, you have a Rack app!
___
## 3. Get blog on Heroku
Another file for Heroku to know how to run your app. Make a Procfile file and put the following in it:
```
web: bundle exec puma -p $PORT config.ru
```

Then run the following commands:

```
git init
heroku create
git add .
git commit -m 'First commit'
git push heroku master
heroku open
```

[^1]: Guide compiled using: 
https://www.clairecodes.com/blog/2015-09-20-deploying-jekyll-on-heroku/
