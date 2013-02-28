---
layout: post
title: "Building my first data-driven Sinatra app and deploying it to Heroku"
date: 2013-02-25 00:06
comments: true
categories: [Flatiron School, technical, Ruby, Sinatra, Heroku, HTML5 boilerplate, Postgres]
---
<div class='container'>
  <p>We just wrapped up our third week at the <a href="http://www.flatironschool.com">Flatiron School</a>. It's been intense, exhausting, but at the same time insanely gratifying. One of the things that kept us all sane is our weekly tradition called 'Feeling Friday', in which we collectively share what's on our mind.</p>

  <p>That being said, I noticed that 1) some people get somewhat uncomfortable when they have to open up in public, and 2) others were scrambling to get their thoughts together, when being put on the spot. I've been in the second category before. The 'Feeling Friday Web App' should come in handy to make this whole experience even more awesome by allowing anonymous sharing whenever you're up for it. At the same time it's a great way to learn about data-driven Sinatra web apps using the DataMapper gem, and deploying them to Heroku. Both of which I had never done before up until 24h ago.</p>

  <p>Before moving any further, if you got cold shivers reading the word Sinatra or DataMapper, I suggest you read both <a href="http://net.tutsplus.com/tutorials/ruby/singing-with-sinatra/">this</a> and <a href="http://www.blacktm.com/talks/building_web_apps_with_rack_and_sinatra">this</a> tutorial first.</p>

  <p>Let's get cracking. Here's what we'll use to build our app:</p>

  <ul>
    <li>Ruby</li>
    <li>Rack</li>
    <li>Sinatra</li>
    <li>DataMapper gem</li>
    <li>PostgreSQL</li>
    <li>HTML/CSS</li>
    <li>Git (for version control)</li>
    <li>Heroku</li>    
  </ul>

  <h3>Easy does it</h3>
  <p>Let's start it off simple. We first create a new folder and add the brains of the operation, i.e. <code>feelingfriday.rb</code>. Here are the gems we'll be using:</p>

    {% codeblock Gems to include in feelingfriday.rb lang:ruby %}
    require 'rubygems'
    require 'sinatra'
    require 'data_mapper'
    require 'dm-postgres-adapter'{% endcodeblock %}

  <h3>Build your model and Setup the Database</h3>

    {% codeblock Set up Database in feelingfriday.rb lang:ruby %}

      DataMapper.setup(:default, ENV['DATABASE_URL'] || 'postgres://localhost/mydb')

      class Note
        include DataMapper::Resource

        # setting up a Notes table
        # setting up database schema with 5 columns
        property :id, Serial # serial will auto increment
        property :content, Text, :required => true
        property :complete, Boolean, :required => true, :default => false
        property :created_at, DateTime
        property :updated_at, DateTime
      end

      DataMapper.finalize.auto_upgrade!{% endcodeblock %}

  <h3>Setting up the Home Page</h3>
  <p>Once we're done it'll look like this, but let's take it one step at a time.</p>
  <img src="/images/feelingfridayapp.png">

  <p>First</p>

  {% codeblock Get all emotions from the database and display them in the home page lang:ruby %}
      get '/' do
        @notes = Note.all :order => :id.desc
        @title = "All Emotions"
        erb :home
      end{% endcodeblock %}

  {% codeblock Create a view for the home page lang:erb %}
      <section id="add">
      <form action="/" method="post">
        <textarea name="content" placeholder="Your deepest emotions go here&hellip;"></textarea>
        <input type="submit" value="Save emotion">
      </form>
      </section>
      <% @notes.each do |note| %>  
        <article>  
          <p>  
            <%= note.content %>  
            <span><a href="/<%= note.id %>">[edit]</a></span>  
          </p>   
          <p class="meta">Created: <%= note.created_at %></p>  
        </article>
        <br>
      <% end %> 
      <p class="remove">Remove all this Emotional Crap!<a href='/migrate'><input type="submit" name="Submit"></a></p>{% endcodeblock %}

  <h3>Create a layout</h3>

  <h3>Time for some Style</h3>

  <h3>What's Next?</h3>


</div>