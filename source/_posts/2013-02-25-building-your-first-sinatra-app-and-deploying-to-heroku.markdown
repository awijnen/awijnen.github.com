---
layout: post
title: "Building my first data-driven Sinatra app and deploying it to Heroku"
date: 2013-02-25 00:06
comments: true
categories: [Flatiron School, technical, Ruby, Sinatra, Heroku, HTML5 boilerplate, Postgres]
---
<div class='container'>
  <p>We just wrapped up our third week at the <a href="http://www.flatironschool.com">Flatiron School</a>. It's been intense, exhausting, but at the same time insanely gratifying. One of the things that kept us all sane is our weekly tradition called 'Feeling Friday', in which we collectively share what's on our mind.</p>

  <p>I wanted to make the process of sharing emotions in a class room setting somewhat more fun, anonymous, and continuous (as opposed to just once every week). The result is the Feeling Friday app, which you can find on <a href="http://feelingfriday.me">FeelingFriday.me</a>.</p>

  <p>Before moving any further, if you got cold shivers reading the word Sinatra or DataMapper, I suggest you read both <a href="http://net.tutsplus.com/tutorials/ruby/singing-with-sinatra/">this</a> and <a href="http://www.blacktm.com/talks/building_web_apps_with_rack_and_sinatra">this</a> tutorial first.</p>

  <p>Let's get cracking. Feel free to refer to the source code on github <a href="https://github.com/awijnen/Feeling_Friday">here</a>. Here's what we'll use to build our app:</p>

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

  <p>We’re going to be using a PostgreSQL database to store the notes, and we’ll use the gem <a href="http://datamapper.org/">DataMapper</a> to communicate with the database. DataMapper is a gem that packages all the code you need to automatically map the classes you create to entries in the database. For example, we'll create a class Notes, which DataMapper will 'map' to a Notes table in the database. Every time you create a new Notes instance (i.e. you create a note), DataMapper will make sure it saves it to the database. Finally, you'll need to specify an adapter gem, that'll allow DataMapper to talk to the type of database you selected (in our case PostgreSQL).</p>

  <p>Please note that throughout the rest of this post I'll use 'Note' and 'Emotion' interchangeably.</p>

  <h3>Build your model and Setup the Database</h3>

  <p>Let's take advantage of <a href="http://datamapper.org/">DataMapper</a> right away. We'll create the class Note, and Comment, which are the two key building block of our Feeling Friday app, and we'll include DataMapper as a resource. This allows us to specify properties, that'll map to columns in the database table.</p>

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

        has n, :comments
      end

      class Comment
        include DataMapper::Resource

        property :id, Serial
        property :content, Text
        property :created_at, DateTime
        property :updated_at, DateTime

        belongs_to :note
      end

      DataMapper.finalize.auto_upgrade!{% endcodeblock %}

      <p></p>

  <h3>Setting up the Home Page</h3>
  <p>Once we're done it'll look like this, but let's take it one step at a time.</p>
  <img src="/images/feelingfridayapp.png">

  <p>First, let's go into <code>feelingfriday.rb</code> and create the necessary routes to create, read, update, and delete notes and comments</p>

  <h3>Get all emotions</h3>

  {% codeblock Get all emotions from the database and display them in the home page lang:ruby %}
      get '/' do
        @notes = Note.all :order => :id.desc
        @title = "All Emotions"
        erb :home
      end{% endcodeblock %}

  <p>On the second line you see how we retrieve all the notes from the database. If you’ve used ActiveRecord (the ORM used in Rails) before, DataMapper’s syntax will feel very familiar. The notes are assigned to the <code>@notes</code> instance variable. It’s important to use instance variables (that’s variables beginning with an @) so that they’ll be accessible from within the view file.
  </p>
  <p>We set the <code>@title</code> instance variable, and load the views/home.erb view file through the ERB parser.</p>
  <p>Now let's first create the structure of our html page, before we start populating it with emotions.</p>

    <h3>Create a layout</h3>
  <p>It's time to create the html struture of our page. We'll do that in two parts. The first part is the skeleton of the website, which will basically be everything that's copied across pages (i.e. header, footer, background, etcera. This is typically stored in the <code>layout.erb</code> file.</p>

  {% codeblock layout.erb lang:html %}  
    <html lang="en">  
    <head>  
      <meta charset="utf8">  
      <title><%= @title + ' | Feeling Friday' %></title>  
      <link href="/reset.css" rel="stylesheet" type="text/css">  
      <link href="/style.css" rel="stylesheet" type="text/css">
      <script src="/main.js"></script>
    </head>  
    <body>  
      <header>  
        <hgroup>  
          <h1><a href="/">Feeling Friday</a></h1>  
          <h2>'cause we're an emotional bunch</h2>
          <a id = "github" href="https://github.com/awijnen/Feeling_Friday">
            <img src= "https://s3.amazonaws.com/github/ribbons/forkme_right_gray_6d6d6d.png" alt="Fork me on GitHub">
          </a>   
        </hgroup>  
      </header>  
      
        <div id="main">
          <%= yield %>  
        </div>  
        
      <footer>  
        <p><small>An app for the <a href="http://students.flatironschool.com/">Students of the Flatiron School</a>.</small></p> 
      </footer> 

      # Google Analytics
      <script type="text/javascript">

        var _gaq = _gaq || [];
        _gaq.push(['_setAccount', 'UA-38874027-1']);
        _gaq.push(['_trackPageview']);

        (function() {
          var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
          ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
          var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
        })();

      </script>

    </body>  
    </html>{% endcodeblock %}

  <p>The most important line of code in <code>layout.erb</code> is where <code>yield</code> is defined. It is at this exact moment that we'll insert all the html content defined in <code>home.erb</code>, <code>edit.erb</code>, and <code>delete.erb</code> (and any other view you might want to add for that matter).</p>

  <p>Next, we'll code up the html for the actual emotion form (where you add a new emotion), as well as the html that will display all the emotions from the database. Again, this code will be injected in <code>layout.erb</code> where <code>yield</code> is defined.</p>

  {% codeblock home.erb lang:erb %}
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

  <p>We'll continue now by adding in the routes to edit, and delete emotions, as well as create views for edit, and delete respectively.</p>

  <h3>Create an Emotion</h3>
    <p>Right now if you try submitting the form on the home page, you’re going to get a route error. Let’s create the POST route for the home page now:</p>
    
    {% codeblock Create a new emotion and save it to the database lang:ruby %}
      post '/post' do
        n = Note.new
        n.content = params[:content]
        n.created_at = Time.now
        n.updated_at = Time.now
        n.save
        redirect '/'
      end{% endcodeblock %}

    <p>So when a post request is made on the homepage, we create a new Note object in n (thanks to the DataMapper ORM, Note.new represents a new row in the notes table in the database). The content field is set to the submitted data from the textarea and the created_at and updated_at datetime fields are set to the current timestamp.</p>
    
    <p>The new note is then saved, and the user redirected back to the homepage where the new note will be displayed.</p>

  <h3>Update an emotion</h3>
   {% codeblock Before we can update an emotion we have to fetch it first lang:ruby %}
    get '/:id' do  
      @note = Note.get params[:id]  
      @title = "Edit note ##{params[:id]}"  
      erb :edit  
    end{% endcodeblock %}

   {% codeblock edit.erb: The view for editing a note lang:ruby %}
    <% if @note %>  
      <form action="/<%= @note.id %>" method="post" id="edit">  
        <input type="hidden" name="_method" value="put">  
        <textarea name="content"><%= @note.content %></textarea>   
        <input type="submit">  
      </form>  
      <p><a href="/<%= @note.id %>/delete">Delete</a></p>  
    <% else %>  
      <p>Note not found.</p>  
    <% end %>{% endcodeblock %}    

    {% codeblock Update an existing emotion and save it to the database lang:ruby %}
      put '/:id' do  
        n = Note.get params[:id]  
        n.content = params[:content]  
        n.complete = params[:complete] ? 1 : 0  
        n.updated_at = Time.now  
        n.save  
        redirect '/'  
      end{% endcodeblock %}

  <h3>Delete an emotion</h3>
    {% codeblock Before we can delete an emotion we have to fetch it first lang:ruby %}
     get '/:id/delete' do  
      @note = Note.get params[:id]  
      @title = "Confirm deletion of note ##{params[:id]}"  
      erb :delete  
    end{% endcodeblock %}

    {% codeblock delete.erb: The view for deleting a note lang:erb %}
    <% if @note %>  
      <p>Are you sure you want to delete the following note: <em>"<%= @note.content %>"</em>?</p> 
      <form action="/<%= @note.id %>" method="post">  
        <input type="hidden" name="_method" value="delete">  
        <input type="submit" value="Yes, Delete It!">  
        <a href="/<%= @note.id %>">Cancel</a>  
      </form>  
    <% else %>  
      <p>Note not found.</p>  
    <% end %>{% endcodeblock %} 

    {% codeblock Delete an existing emotion, which removes it from the database lang:ruby %}
    delete '/:id' do  
      n = Note.get params[:id]  
      n.destroy  
      redirect '/'  
    end {% endcodeblock %}

  <h3>Time for some Style</h3>
  <p>Our routes and views are all set. Time to make the whole thing look pretty. I won't go into detail of the css. I used HTML5 boilerplate to reset any cross-browser quirks, and wrote some custom css to give the whole thing a nice NYC Flatiron district feel. Hope you like it.</p>

  {% codeblock style.css lang:css %}
    @font-face {
      font-family: 'Raleway';
      font-style: normal;
      font-weight: 200;
      src: local('Raleway ExtraLight'), local('Raleway-ExtraLight'), url(http://themes.googleusercontent.com/static/fonts/raleway/v6/8KhZd3VQBtXTAznvKjw-k73hpw3pgy2gAi-Ip7WPMi0.woff) format('woff');
    }

    body {  
        font-family: 'Raleway';
        font-size: 11pt;
        margin: 35px auto;  
        width: 700px;
        background:url('/images/newyork.jpg') no-repeat fixed;
    }  

    header {  
        text-align: center;  
        margin: 0 0 20px;  
    } 

    header a#github{
        position: fixed; top: 0;
        right: 0;
        border: 0;
    } 

    img {
      max-width: 100%;
    }

    header h1 {  
        display: inline;  
        font-size: 32px;  
    }  
    header h1 a:link, header h1 a:visited {  
        color: #444;  
        text-decoration: none;  
    }  
    header h2 {  
        font-size: 16px;
        font-weight: normal;
        font-style: italic;  
        color: #999;  
    }  
    #main {  
        margin: 0 0 20px;
        padding: 20px 20px 20px 20px;
        background-color: #ddd;  
    }  
    #add {  
        margin: 0 0 20px;  
    }  
    #add textarea {  
        height: 28px; 
        width: 534px;  
        max-width: 534px;  
        padding: 10px;  
        border: 1px solid #ddd;  
        background-color: #fafafa;
    }  
    #add input {  
        height: 50px;  
        width: 100px;  
        margin: -41px 0 0;  
        border: 1px solid #ddd;  
        background: #fafafa;  
    }  
    p.remove {
        display: none;
    }

    p.remove a {
        margin-left: 5px;
    }

    #edit textarea {  
        height: 30px;  
        width: 450px;  
        padding: 10px;  
        border: 1px solid #ddd;  
    }  

    #edit input[type=submit] {  
        height: 50px;  
        width: 100px;  
        margin: -45px 0 0;  
        border: 1px solid #ddd;  
        background: white;  
    }  
    #edit input[type=checkbox] {  
        height: 50px;  
        width: 20px;  
    }  
    article {  
        border: 1px solid #eee;   
        padding: 12px 12px 12px 12px; 
        margin: 15px 0 15px 0; 
    }  
    article:first-of-type {  
        border: 1px solid #eee;  
    }  
    article:nth-child(even) {  
        background: #fafafa;  
    }
    article:nth-child(odd) {  
        background: #fafafa;  
    }
      
    article span {  
        font-size: 0.8em;  
    }  
    article .comments{
        margin-top:11px;
    }
    p {  
        margin: 0 0 5px;  
    } 

    p.note-content{
        font-size: 1.1em;
    } 

    p.comment-content{
        font-size : 1em;
    }
    .meta {  
        font-size: 0.8em;  
        font-style: italic;  
        color: #888;  
    }  
    .links {  
        font-size: 1.8em;  
        line-height: 0.8em;  
        float: rightright;  
        margin: -10px 0 0;  
    }  
    .links a {  
        display: block;  
        text-decoration: none;  
    }
    form textarea, form input{
        font-family: 'Raleway';
    }
    form input[type="submit"]{
        font-size:0.9em;
    }
    .form-toggle {
        cursor: pointer;
        font-size: 1em;
    }

    .comments {
        background: #eee;
        padding: 15px;
    }

    .comments .comment{
        border-bottom: 1px dotted #ddd ;
        margin: 0 0 10px 0;
        padding: 0 0 5px 0;
    }

    .comments form {
        padding: 10px 0;
    }

    .comments form input {
        border: none;
    }

    .comments form button {
        padding: 2px;
    }

    footer {
        background-color: #ddd;
        padding: 10px 10px;
    }

  {% endcodeblock %}

  <h3>Conclusion</h3>
  <p>There you have it, you built your first data driven Sinatra app. Awesome! Hope you enjoyed it. Don't hesitate to reach out on <a href="https://www.twitter.com/anthonywijnen">twitter</a>, or <a href="http://github.com/awijnen">github</a> if you have any questions.</p>

</div>