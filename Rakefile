task :default => [:serve]

desc "Build the site"
task :build do
  system 'bundle exec jekyll build'
end

desc "Serve and watch the site in development mode"
task :serve do
  system 'bundle exec guard'
  system 'bundle exec jekyll serve --watch --port 3000'
end

namespace :post do
  desc "Create a new post"
  task :create, [:title] do |t, args|
    puts args
    date = Time.now
    title = args[:title]
    puts "Creating post \"#{title}\""

    content = <<-EOS.gsub(/^+/,'')
---
layout: post
date: #{date.strftime('%Y-%m-%d %H:%M:%S')}
title: #{title}
published: false
---
EOS

    MAX_DISPLAYED_TITLE = 6
    datef = date.strftime('%Y-%m-%d')
    titlef = title.split(' ').take(6).join("-")
    filename = "_posts/#{datef}-#{titlef}.md"

    puts "Writing to file #{filename}"
    File.write(filename, content)
  end
end
