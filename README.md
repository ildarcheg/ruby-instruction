# ruby-instruction

## MODULE 1
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
### Controllers and routes 
#### Foo postgresql
Add controller without defaults rspec and remove model spec
```
rails-api g scaffold Foo name --orm active_record --no-request-specs --no-routing-specs --no-controller-specs
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
Check controller and comment defaults renders 
```
class FoosController < ApplicationController
  before_action :set_foo, only: [:show, :update, :destroy]

  def index
    @foos = Foo.all
    #render json: @foos
  end

  def show
    render json: @foo
  end

  def create
    @foo = Foo.new(foo_params)

    if @foo.save
      #render json: @foo, status: :created, location: @foo
      render :show, status: :created, location: @foo
    else
      render json: @foo.errors, status: :unprocessable_entity
    end
  end

  def update
    if @foo.update(foo_params)
      head :no_content
    else
      render json: @foo.errors, status: :unprocessable_entity
    end
  end

  def destroy
    @foo.destroy

    head :no_content
  end

  private

    def set_foo
      @foo = Foo.find(params[:id])
    end

    def foo_params
      params.require(:foo).permit(:name)
    end
end
```
Add API route to routes
```
Rails.application.routes.draw do
  scope :api defaults: {format: :json} do 
    resources :foos, except: [:new, :edit]
  end   
end
```
Run test for RDBMS
```
rspec -e RDBMS -fd
```
Create first object in DB, run server and check in browser localhost:3000/api/foos
```
rails c
Foo.create(:name=>"test")
exit
rails s
```
#### Bar mongodb
Add mongoid to Gemfile, run bundle ang mongoid config (create mongoid.yml)
```
#gem 'mongoid', '~>5.1', '>=5.1.5'
bundle 
rails g mongoid:config
```
Update mongoid.yml with
```
development:
  clients:
    default:
      database: myapp_development
      hosts:
        - <%= ENV['MONGO_HOST'] ||= "localhost:27017" %>
      options:
  options:
test:
  clients:
    default:
      database: myapp_test
      hosts:
        - <%= ENV['MONGO_HOST'] ||= "localhost:27017" %>
      options:
        read:
          mode: :primary
        max_pool_size: 1
production:
  clients:
    default:
      uri: <%= ENV['MLAB_URI'] %>
      options:
        connect_timeout: 15
```
Update config/application.rb with
```
 Mongoid.load!('./config/mongoid.yml')
 #which default ORM are we using with scaffold
 #add  --orm mongoid, or active_record 
 #    to rails generate cmd line to be specific
 config.generators {|g| g.orm :active_record}
 #config.generators {|g| g.orm :mongoid}
````
Add controller without defaults rspec and remove model spec
```
rails-api g scaffold Bar name --orm mongoid --no-request-specs --no-routing-specs --no-controller-specs
rm spec/model/bar_spec.rb
```
Update controller with
```
class BarsController < ApplicationController
  before_action :set_bar, only: [:show, :update, :destroy]

  def index
    @bars = Bar.all
    #render json: @bars
  end

  def show
    #render json: @bar
  end

  def create
    @bar = Bar.new(bar_params)

    if @bar.save
      #render json: @bar, status: :created, location: @bar
      render :show, status: :created, location: @bar
    else
      render json: @bar.errors, status: :unprocessable_entity
    end
  end

  def update
    if @bar.update(bar_params)
      head :no_content
    else
      render json: @bar.errors, status: :unprocessable_entity
    end
  end

  def destroy
    @bar.destroy

    head :no_content
  end

  private

    def set_bar
      @bar = Bar.find(params[:id])
    end

    def bar_params
      params.require(:bar).permit(:name)
    end
end
```
Update Bar model with
```
class Bar
  include Mongoid::Document
  include Mongoid::Timestamps
  field :name, type: String
end
```
Add API route to routes
```
Rails.application.routes.draw do
  scope :api defaults: {format: :json} do 
    resources :foos, except: [:new, :edit]
    resources :bars, except: [:new, :edit]
  end   
end
```
Update app/views/bar/_bar.json.jbuilder with
```
#json.extract! bar, :id, :name, :created_at, :updated_at

json.id bar.id.to_s
json.name bar.name
json.created_at bar.created_at
json.updated_at bar.updated_at

json.url bar_url(bar, format: :json)
```
Update test with
```
require 'rails_helper'

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
      expect(parsed_body).to include("created_at")
      expect(parsed_body).to include("id"=>object.id.to_s)
    end
  end
end
````
Run test for MongoDB
```
rspec -e MongoDB -fd
```
Create few instances on MongoDB
```
rails c
pp (1..3).each.map { |idx| Bar.create(:name=>"test#{idx}") }; nil
quit
```
Check in browser localhost:3000/api/bars
```
rails s
```
### HTTP 
Update Gemfile (dev and test) with
```
gem 'httparty', '~>0.14', '>=0.14.0'
```
Update config/application.rb for CORS with
```
# update Gemfile with gem 'rack-cors', '~>0.4', '>=0.4.0', :require => 'rack/cors'
config.middleware.insert_before 0, "Rack::Cors" do
  allow do
    origins '*'
    #origins 'siteB.com' 

    resource '/api/*', 
      :headers => :any, 
      :methods => [:get, :post, :put, :delete, :options]
  end
end
``` 
###Additional setup
Update Gemfile for Puma browser with 
```
gem 'puma', '~>3.6', '>=3.6.0', :platforms=>:ruby
```
Update Gemfile for debugging with and use 'byebug' inside the code
```
gem 'byebug', '~>9.0', '>=9.0.6' # for dev and test
```
Update Gemfile for advanced console and debuggins with 
```
gem 'pry-rails', '~>0.3', '>=0.3.4'
gem 'pry-byebug', '~>3.4', '>=3.4.0' # for dev and test
```
### HEROKU
Update Gemfile with 
```
group :production do
  gem 'rails_12factor', '~>0.0', '>= 0.0.3'
end
```
### UI
Create UI 
```
rails g controller ui
rm spec/controllers/ui_controller_spec.rb
mkdir echo "Capstone Site Under Construction -- check back soon" > app/views/ui/index.html.erb
```
Update routes with
```
get '/ui' => 'ui#index'
get '/ui#' => 'ui#index'   
root "ui#index"
```
Install heroku
```
brew install heroku/brew/heroku
```
Create app with remote rep named 'staging' and 'production'
```
heroku create AppName-staging --remote staging
heroku create AppName-production --remote production
git remote -v
```
Create a branch for staging
```
git checkout -b staging-app
git branch
```
Push local 'staging-app' branch to 'staging' remote rep 'master' branch
```
git push staging staging-app:master
```
Check logs for errors
```
heroku logs --remote staging
```
Run migrate for Foos
```
heroku run rake db:migrate --remote staging
```
Check environment variables and setup MLAB_URI
```
heroku config --remote staging
heroku config:set MLAB_URI=mongodb://<dbuser>:<dbpassword>@ds133054.mlab.com:33054/capstone-staging --remote staging
heroku config --remote staging
```
Create Procfile with
```
web: bundle exec puma -C config/puma.rb
```
Turn on ssl on production - config/environments/production.rb
```
config.force_ssl = true
```
### MERGE and PUSH
Merge master with staging-app
```
git checkout master
git merge staging-app
git push production master
```

## MODULE 2
### ASSETS PIPELINE
Get back to master and create asset-pipeline branch
```
git checkout master
git checkout -b asset-pipeline
```
Update Gemfile with 
```
gem 'sass-rails', '~> 5.0', '>=3.4.22'
gem 'uglifier', '~> 3.0', '>=3.0.2'
gem 'coffee-rails', '~> 4.1', '>= 4.1.0'
gem 'jquery-rails', '~>4.2', '>=4.2.1'

source 'https://rails-assets.org' do
  gem 'rails-assets-bootstrap', '~>3.3', '>= 3.3.7'
  gem 'rails-assets-angular', '~>1.5', '>= 1.5.8'
  gem 'rails-assets-angular-ui-router', '~>0.3', '>= 0.3.1'
  gem 'rails-assets-angular-resource', '~>1.5', '>= 1.5.8'
end
```
Run bundle
```
bundle
```
Create additional folder for assets in app/assets
```
mkdir app/assets/stylesheets
mkdir app/assets/javascripts
```
Creare CSS manifest and JS manifest
```
echo -e "/*\n * SPA Demo Stylesheet Manifest file\n */" >> app/assets/stylesheets/spa-demo.css
echo -e "// SPA Demo Javascript Manifest File" >> app/assets/javascripts/spa-demo.js
```
Create initializer config file
```
echo -e "Rails.application.config.assets.version = '1.0'" >> config/initializers/assets.rb
echo -e "Rails.application.config.assets.precompile += %w( spa-demo.js spa-demo.css )" >> config/initializers/assets.rb
```
Update ui view in app/views/ui/index.html.erb with
```
<!DOCTYPE html>
<html lang="en" ng-app="spa-demo">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <%= stylesheet_link_tag    "spa-demo", :media => "all" %>
    <%= javascript_include_tag "spa-demo" %>
  </head>
  <body>

    <div class="container">
      <h1>Hello</h1>
      <span>(from app/views/ui/index.html.erb)</span>
      <div ui-view></div>
    </div>

  </body>
</html>
```
Add boostrap to CSS assets
```
/*
 * SPA Demo Stylesheet Manifest File
 *
 *= require bootstrap
 */
```
Add scripts to JS assets
```
// SPA Demo Javascript Manifest File
//= require jquery2
//= require bootstrap
//= require angular
//= require angular-ui-router
//= require angular-resource
```
Run server and check in browser that it is in place
```
rails s
```
Define an application adding ng-app and check in browser for error
```
<html lang="en" ng-app="spa-demo">
...
<div ui-view></div>div>
```
Deploy to staging hosting and check for error
```
git push staging asset-pipeline:master
```
### CREATING SPA-APP
Add folders as following
```
mkdir app/assets/javascripts/spa-demo
mkdir app/assets/javascripts/spa-demo/pages
```
Create app module at app/assets/javascripts/spa-demo/app.module.js
```
(function() {
  "use strict";

  angular
    .module("spa-demo", ["ui.router"]);

})();
```
Create simple page at app/assets/javascripts/spa-demo/pages/main/html
```
<div>
  <h2>Sample App</h2>
  <span>(from spa-demo/pages/main.html)</span>
  <sd-foos></sd-foos>
</div>
```
Create constant at app/assets/javascripts/spa-demo/app.constant.js.erb
```
(function() {
  "use strict";

  angular
    .module("spa-demo")
    .constant("spa-demo.APP_CONFIG", {

      main_page_html: "<%= asset_path('spa-demo/pages/main.html') %>"

    });

})();
```
Create router at app/assets/javascripts/spa-demo/app.router.js
```
(function() {
  "use strict";

  angular
    .module("spa-demo")
    .config(RouterFunction);

  RouterFunction.$inject = ["$stateProvider",
                            "$urlRouterProvider", 
                            "spa-demo.APP_CONFIG"];

  function RouterFunction($stateProvider, $urlRouterProvider, APP_CONFIG) {
    $stateProvider
    .state("home",{
      url: "/",
      templateUrl: APP_CONFIG.main_page_html,
      // controller: ,
      // controllerAs: ,
    })

    $urlRouterProvider.otherwise("/");
  }
})();
```
Add our scripts to app manifest app/assets/javascripts/spa-demo.js
```
//= require spa-demo/app.module
//= require spa-demo/app.router
//= require spa-demo/app.constant
```
Create a branch foo-ui?
```
create
```
Add server_url to router.js
```
server_url: "<%= ENV['RAILS_API_URL'] %>",
```
Create 'foos' folder and 'foos.module' in spa-demo
```
(function() {
  "use strict";

  angular
    .module("spa-demo.foos", ["ngResource"]);

})();
```
Add script to the manifest
```
//= require spa-demo/foos/foos.module
```
Add a dependensy to app.module.js
```
  angular
    .module("spa-demo", [
      "ui.router",
      "spa-demo.foos"
    ]);
```
Create a factory for spa-demo.foos module at app/assets/javascripts/spa-demo/foos/foos.service.js
```
(function() {
  "use strict";

  angular
    .module("spa-demo.foos")
    .factory("spa-demo.foos.Foo", FooFactory);

  FooFactory.$inject = ["$resource", "spa-demo.APP_CONFIG"];
  function FooFactory($resource, APP_CONFIG) {
    return $resource(APP_CONFIG.server_url + "/api/foos/:id",
      { id: '@id'},
      { update: { method: "PUT" }
      }
      );
  }
})();
```
Add service to the manifest
```
//= require spa-demo/foos/foos.service
```
Create a controller for foos 
```
(function() {
  "use strict";

  angular
    .module("spa-demo.foos")
    .controller("spa-demo.foos.FoosController", FoosController);

  FoosController.$inject = ["spa-demo.foos.Foo"];

  function FoosController(Foo) {
      var vm = this;
      vm.foos;
      vm.foo;

      activate();
      return;
      ////////////////
      function activate() {
        newFoo();
      }

      function newFoo() {
        vm.foo = new Foo();
      }
      function handleError(response) {
        console.log(response);
      } 
      function edit(object, index) {
      }
      function create() {
      }
      function update() {
      }
      function remove() {
      }
      function removeElement(elements, element) {
      }      
  }
})();
```
Add service to the manifest
```
//= require spa-demo/foos/foos.controller
```
Create simple html for foos in foos folder
```
<div>
  <h3>Foos</h3>
  <span>(from spa-demo/foos/foos.html)</span>

</div>
```
Create a directive foos for previous html 
```
(function() {
  "use strict";

  angular
    .module("spa-demo.foos")
    .directive("sdFoos", FoosDirective);

  FoosDirective.$inject = ["spa-demo.APP_CONFIG"];

  function FoosDirective(APP_CONFIG) {
    var directive = {
        templateUrl: APP_CONFIG.foos_html,
        replace: true,
        bindToController: true,
        controller: "spa-demo.foos.FoosController",
        controllerAs: "foosVM",
        restrict: "E",
        scope: {},
        link: link
    };
    return directive;

    function link(scope, element, attrs) {
      console.log("FoosDirective", scope);
    }
  }

})();
```
Update constant with foos_html
```
foos_html: "<%= asset_path('spa-demo/foos/foos.html') %>"
```
Add directive to the manifest
```
//= require spa-demo/foos/foos.directive
```
Update main page in pages folder
```
<sd-foos></sd-foos>
```
Add folder spa-demo in assets/stylesheets and layout.css
```
/* 
 * layout.css
 */

 body {
  background-color: #d2b48c;
 }
```
Add layout.css to css manifest
```
/*
 * SPA Demo Stylesheet Manifest File
 *
 *= require bootstrap
 *= require spa-demo/layout
 */
```
Create some foos in DB
```
rails c
(1..5).each { |i| Foo.create(:name=>"test#{i}") }
Foo.count
```
Add ng-repeat to foos.html]
```
  <ul>
    <li ng-repeat="foo in foosVM.foos | orderBy:'name'">
      <a ng-click="foosVM.edit(foo)">{{foo.name}}</a>
    </li>
  </ul>
```
Add form to foos.html
```
  <form>
    <div>
      <label>Name:</label>
      <input name="name"
              ng-model="foosVM.foo.name"
              required="required"/>
    </div>

    <button ng-if="!foosVM.foo.id" 
             type="submit"
             ng-click="foosVM.create()">Create Foo</button>  
    <div ng-if="foosVM.foo.id">
      <button type="submit"
              ng-click="foosVM.update()">Update Foo</button>
      <button type="submit"
              ng-click="foosVM.remove()">Delete Foo</button>
    </div>               
  </form>
```
Update FooService - FooFactory and new function
```
  function FooFactory($resource, APP_CONFIG) {
    return $resource(APP_CONFIG.server_url + "/api/foos/:id",
      { id: '@id'},
      { 
        update: { method: "PUT",
                  transformRequest: buildNestedBody },
        save: { method: "POST",
                  transformRequest: buildNestedBody }
      }
      );
  }

  //nests the default payload below a "foo" element 
  //as required by default by Rails API resources
  function buildNestedBody(data) {
   return angular.toJson({foo: data})
  }  
```
Update foo controller
```
(function() {
  "use strict";

  angular
    .module("spa-demo.foos")
    .controller("spa-demo.foos.FoosController", FoosController);

  FoosController.$inject = ["spa-demo.foos.Foo"];

  function FoosController(Foo) {
      var vm = this;
      vm.foos;
      vm.foo;
      vm.edit   = edit;
      vm.create = create;
      vm.update = update;
      vm.remove = remove;      

      activate();
      return;
      ////////////////
      function activate() {
        newFoo();
        vm.foos = Foo.query();
      }

      function newFoo() {
        vm.foo = new Foo();
      }
      function handleError(response) {
        console.log(response);
      } 
      function edit(object) {
        console.log("selected", object);
        vm.foo = object;        
      }

      function create() {
        //console.log("creating foo", vm.foo);
        vm.foo.$save()
          .then(function(response){
            //console.log(response);
            vm.foos.push(vm.foo);
            newFoo();
          })
          .catch(handleError);        
      }

      function update() {
        //console.log("update", vm.foo);
        vm.foo.$update()
          .then(function(response){
            //console.log(response);
        })
        .catch(handleError);        
      }

      function remove() {
        //console.log("remove", vm.foo);
        vm.foo.$delete()
          .then(function(response){
            //console.log(response);
            //remove the element from local array
            removeElement(vm.foos, vm.foo);
            //vm.foos = Foo.query();
            //replace edit area with prototype instance
            newFoo();
          })
          .catch(handleError);                
      }


      function removeElement(elements, element) {
        for (var i=0; i<elements.length; i++) {
          if (elements[i].id == element.id) {
            elements.splice(i,1);
            break;
          }        
        }        
      }      
  }
})();
```
