require "rubygems"
require "bundler/setup"
require "stringex"
require 'ruby-pinyin'

deploy_branch  = "master"
deploy_default = "push"

## -- Misc Configs -- ##

public_dir      = "_site"    # compiled site directory
source_dir      = "."   # source file directory
blog_index_dir  = '.'    # directory for your blog's index page (if you put your index in source/blog/index.html, set this to 'source/blog')
deploy_dir      = "_deploy"   # deploy directory (for Github pages deployment)
stash_dir       = "_stash"    # directory to stash posts for speedy generation
posts_dir       = "_posts"    # directory for blog files
themes_dir      = ".themes"   # directory for blog files
new_post_ext    = "markdown"  # default new post file extension when using the new_post task
new_page_ext    = "markdown"  # default new page file extension when using the new_page task
server_port     = "4000"      # port for preview server eg. localhost:4000


# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{source_dir}/#{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
     title = get_stdin("Enter a title for your post: ")
  end
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  mkdir_p "#{source_dir}/#{posts_dir}"
  #args.with_defaults(:title => 'new-post')
  #title = args.title
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{PinYin.permlink(title)}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "category: tech"
    post.puts "tags: "
    post.puts "---"
    post.puts ""
    post.puts "<!--more-->"
    post.puts ""
    post.puts "enjoy!"
  end
end

def ok_failed(condition)
  if (condition)
    puts "OK"
  else
    puts "FAILED"
  end
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end
