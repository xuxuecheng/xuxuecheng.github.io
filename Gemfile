source "https://rubygems.org"

# Ruby 版本固定，保证本地与 Cloudflare Pages 构建环境一致
ruby "3.1.6"

gem "jekyll", "~> 4.3.4"

group :jekyll_plugins do
  gem "jekyll-sitemap"
  gem "jekyll-feed"
end

# Windows / JRuby 兼容（官方提示需要的平台依赖）
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", platforms: [:mingw, :x64_mingw, :mswin]
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]
