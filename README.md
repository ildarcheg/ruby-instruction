# ruby-instruction

### Install Ruby

Install Brew
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew --version
```
Install Rbenv
```
brew install rbenv ruby-build
echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bash_profile
source ~/.bash_profile
rbenv --version
```
Install Ruby 2.3.3
```
rbenv install 2.3.3
rbenv global 2.3.3
ruby --version
```
Install GIT
```
brew install git
source ~/.bash_profile
git --version
git config --global color.ui true
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR@EMAIL.com"
```
Install PostgreSQL
```
brew install postgresql
brew services start postgresql
brew services list
```
Install MongoDB
```
brew install mongodb
brew services start mongodb
brew services list
```
Install Node.js
```
brew install node
node --version
```
Install ImageMagick
```
brew install imagemagick@6
#check messages and get next line from them
echo 'export PATH="$PATH:/usr/local/opt/imagemagick@6/bin"' >> ~/.bash_profile
source ~/.bash_profile
mogrify --version
```
Install PhantomJS
```
brew install phantomjs
phantomjs --version
```
Install ChromeDriver
```
brew install chromedriver
chromedriver --version
```
Install Sublime Text 3
```
brew install sublime-text
subl --version
```
Install rails gem
```
gem install rails -v 4.2.6 --no-ri --no-doc
rbenv rehash
rails -v
```
Install the rails-api gem
```
gem install rails-api -v 0.4.0 --no-ri --no-doc
rails-api -v
gem uninstall railties -v 5.0.1
rbenv rehash
rails-api -v
```
Install the bundler
```
gem install bundler --no-ri --no-doc
```
Check
```
git clone https://github.com/jhu-ep-coursera/capstone_demoapp.git module1
cd module1
git checkout module2.start
git checkout -b module1
bundle
bundle exec rake db:create
bundle exec rake db:migrate
bundle exec rake db:migrate RAILS_ENV=test
bundle exec rake 
```

### Setup app

Create app folder and init git
```
cd
mkdir capstone_demoapp
git init .
echo "# Capstone Demo Application" > README.md
git add README.md
git commit -m "initial commit"
```
Create app in current directory (use 'new .', -T - no test)
```
rails-api new . -T -d postgresql
```
Adjust database.yml
```
default: &default
  adapter: postgresql
  encoding: unicode
  pool: 5
  username: <%= ENV['POSTGRES_USER'] %>
  password: <%= ENV['POSTGRES_PASSWORD'] %>
  host: <%= ENV['POSTGRES_HOST'] %>

development:
  <<: *default
  database: capstone_demoapp_development

test:
  <<: *default
  database: capstone_demoapp_test

production:
  adapter: postgresql
  encoding: unicode
  pool: 5
  url: <%= ENV['DATABASE_URL'] %>
```
Setup Gemfile
```
# 
source 'https://rubygems.org'

gem 'rails', '4.2.6'
gem 'rails-api', '~>0.4', '>=0.4.0'

gem 'jbuilder', '~> 2.0', '>=2.6.0'

group :development do
  gem 'spring', '~>2.0', '>=2.0.0'
end

group :development, :test do
  gem 'rspec-rails', '~> 3.5', '>=3.5.2'
end

gem 'pg', '0.20.0'
 ```
Rund Bundle
```
bundle
```
Add to app/controller/application_controller.rb
```
class ApplicationController < ActionController::API
  #make the connection between controller action and associated view
  include ActionController::ImplicitRender
end
```
Run rails server and check
```
rails s
```
Install Rspec
```
rails g rspec:install
```
Generate test
```
rails g rspec:request APIDevelopment
```
Add to test file
```
require 'rails_helper'

RSpec.describe "ApiDevelopments", type: :request do
  describe "RDBMS-backed" do
    it "create RDBMS-backed model"
    it "expose RDBMS-backed API resource"
  end

  describe "MongoDB-backed" do
    it "create MongoDB-backed model"
    it "expose MongoDB-backed API resource"
  end
end
```
Check the test
```
rspec -fg # with formal documentation
```
Update test with
```require 'rails_helper'

RSpec.describe "ApiDevelopments", type: :request do
  def parsed_body
    JSON.parse(response.body)
  end

  describe "RDBMS-backed" do
    before(:each) { Foo.delete_all }
    after(:each)  { Foo.delete_all }

    it "create RDBMS-backed model" do
      object=Foo.create(:name=>"test")
      expect(Foo.find(object.id).name).to eq("test")
    end

    it "expose RDBMS-backed API resource" do
      object=Foo.create(:name=>"test")
      expect(foos_path).to eq("/api/foos")
      get foo_path(object.id)
      expect(response).to have_http_status(:ok)
      expect(parsed_body["name"]).to eq("test")
    end
  end

  describe "MongoDB-backed" do
    before(:each) { Bar.delete_all }
    after(:each)  { Bar.delete_all }

    it "create MongoDB-backed model" do
      object=Bar.create(:name=>"test")
      expect(Bar.find(object.id).name).to eq("test")
    end
    
    it "expose MongoDB-backed API resource" do
      object=Bar.create(:name=>"test")
      expect(bars_path).to eq("/api/bars")
      get bar_path(object.id) 
      expect(response).to have_http_status(:ok)
      expect(parsed_body["name"]).to eq("test")
    end
  end
end
```

### Controllers

Add controller without defaults rspec and remove model spec
```
rails-api g scaffold Foo name --orm active-record --no-request-specs --no-routing-specs --no-controller-specs
rm spec/model/foo_spec.rb
```
Create a table for Foo model, perform migrate
```
rake db:migrate
```
Check DB (go to db shell)
```
rails db
\d # list of tables
\d foos # table structure
\q # exit
```
Run rspec test
``` 
rspec - e RDBMS -fd
```
