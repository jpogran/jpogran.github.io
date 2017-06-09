source "https://rubygems.org"

gem "rake"
gem "github-pages", group: :jekyll_plugins 
gem "minimal-mistakes-jekyll", :github => "mmistakes/minimal-mistakes"

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-github-metadata"
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
end

# Evaluate Gemfile.local if it exists
if File.exists? "#{__FILE__}.local"
  eval(File.read("#{__FILE__}.local"), binding)
end