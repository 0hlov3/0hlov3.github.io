# 0hlov3.github.io

## Blog-Site
This is the BLOG of https://maxxblow.de

## Jekyll-Uno
A Jekyll Theme, based on the Uno-Theme.
The site is in Progress.

## Install and Test
### Install ruby and other requirements:
```
sudo apt install ruby rubygems gcc g++ make
```
or
```
sudo apt install ruby-full build-essential zlib1g-dev
```

### Install RubyGems Packages:
```
sudo gem install bundler jekyll
```
better solution, edit your Bashrc to:
```
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
and then install the packages"
```
gem install bundler jekyll
```

### Install Dependencies for this repo
```
cd 0hlov3.github.io

bundle install
```

### Serving
```
bundle exec jekyll serve --watch
```

If you would like to run without using the github-pages gem, update your Gemfile to the following:

```
source 'https://rubygems.org'
gem 'jekyll-paginate'
gem 'jekyll-watch'
gem 'kramdown'
gem 'kramdown-parser-gfm'
```