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
