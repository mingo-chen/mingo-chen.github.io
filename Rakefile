require "rubygems"
require 'rake'
require 'yaml'
require 'time'

SOURCE = "."
CONFIG = {
  'version' => "12.3.2",
  'themes' => File.join(SOURCE, "_includes", "themes"),
  'layouts' => File.join(SOURCE, "_layouts"),
  'posts' => File.join(SOURCE, "_posts"),
  'post_ext' => "md",
  'theme_package_version' => "0.1.0"
}

bg_imgs = ["post-bg-2015.jpg","post-bg-alitrip.jpg","post-bg-halting.jpg","post-bg-infinity.jpg","post-bg-universe.jpg","post-bg-unix-linux.jpg","post-bg-web.jpg"]
bg_img = bg_imgs.sample(1)[0]

# Usage: rake post title="A Title" subtitle="A sub title"
desc "Begin a new post in #{CONFIG['posts']}"
task :post do
  abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])
  title = ENV["title"] || "new-post"
  subtitle = ENV["subtitle"] || "This is a subtitle"
  slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  begin
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
    ctime = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d %H:%M')
  rescue Exception => e
    puts "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
    exit -1
  end
  filename = File.join(CONFIG['posts'], "#{date}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  puts "Creating new post: #{filename}, bg: #{bg_img}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/-/,' ')}\""
    post.puts "subtitle: \"#{subtitle.gsub(/-/,' ')}\""
    post.puts "date: \"#{ctime}\""
    post.puts "author: \"mingo\""
    post.puts "header-img: \"assets/img/#{bg_img}\""
    post.puts "tags: []"
    post.puts "---"
    post.puts ""
    post.puts "## What XXX"
    post.puts "当前领域的背景,难点是什么,引申出本解决方案"
    post.puts "XXX是什么? XXX通过什么思路/思想/理论解决了本问题?"
    post.puts "通过阅读本文能获得什么?"
    post.puts ""
    post.puts "## How XXX"
    post.puts "如何使用? 典型的应用场景是什么?"
    post.puts ""
    post.puts "## Why XXX"
    post.puts "实现原理,架构图,设计图"
    post.puts "当前领域的难点是什么?当前其它同类解决方案是怎么做的?"
    post.puts "为什么XXX是最优/是好的,亮点是什么?"
    post.puts "有无缺点"
    post.puts ""
    post.puts "## Other XXX"
    post.puts "在当前领域的竞品有哪些?分别的亮点?性能数据?如何选型及依据"
    post.puts ""
    post.puts "## 总结"
    post.puts "XX思路/思想/理论的一般推广,是否可以在更多的场景/领域进行扩展"
    post.puts "举一反三,以点及面,触类旁通"
    post.puts ""
    post.puts "## 引用&参考"
  end
end # task :post

desc "Launch preview environment"
task :preview do
  system "jekyll --auto --server"
end # task :preview

#Load custom rake scripts
Dir['_rake/*.rake'].each { |r| load r }
