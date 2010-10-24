desc 'Run all tasks'
task :default => [:commit]

desc 'Commit all files'
task :commit do
	sh "git add _posts; git add _drafts; git commit -a ; git push"
end

desc 'Generate tags page'
task :tags do
  puts "Generating tags..."
  require 'rubygems'
  require 'jekyll'
  include Jekyll::Filters
  
  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')
  site.categories.sort.each do |category, posts|
    html = ''
    html << <<-HTML
---
layout: default
title: Postings filed under "#{category}"
---
    <div id="pageBlurb">
        <h1>Postings filed under "#{category}"</h1>
    </div>
    HTML

    html << '<ul id="posts">'
    posts = posts.sort { |a,b| b.date <=> a.date }
    posts.each do |post|
      post_data = post.to_liquid
      html << <<-HTML
        <li>#{post_data['date'].strftime(fmt="%B %d, %Y")} - <a href="#{post.url}">#{post_data['title']}</a></li>
      HTML
    end
    html << '</ul>'

    File.open("categories/#{category}.html", 'w+') do |file|
      file.puts html
    end
  end
  sh "git add categories"
  puts 'Done.'
end

desc 'Generates tag cloud'
task :tagcloud do
	puts 'Generating tag cloud...'
	require 'rubygems'
	require 'jekyll'
	include Jekyll::Filters

	options = Jekyll.configuration({})
	site = Jekyll::Site.new(options)
	site.read_posts('')

	html =<<-HTML
	HTML

	site.categories.sort.each do |category, posts|
		html << <<-HTML
		HTML
      
		s = posts.count
		font_size = 7 + (s*1.1);
		html << "<a href=\"/categories/#{category}.html\" title=\"Postings tagged #{category}\" style=\"font-size: #{font_size}px; line-height:#{font_size}px\">#{category}</a> "
	end

	File.open('_includes/categories.html', 'w+') do |file|
		file.puts html
	end

	puts 'Done.'
end
