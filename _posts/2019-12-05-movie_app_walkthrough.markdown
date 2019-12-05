---
layout: post
title:      "Movie App Walkthrough"
date:       2019-12-05 20:42:56 +0000
permalink:  movie_app_walkthrough
---


I am super excited for my first project that came from scratch to a complete functional app. What is it about? It is a command line interface project which retrieve information about popular movies from themoviedb.org website. 

I created a file structure based on bundler ruby gem which consist of of the lib and bin files: The lib folder is pretty much the bulk of the files and codes used by application to run its operation, and bin folder are just executable files that used for running the required files and dependancies. 

So, here is the runthrough of my project. First of all, I set up my environment dependancies:



### moviepod/lib/moviepod.rb

This is my environment file where I setup the environment in which I utilize all the tools and files that are playing with each other to make the app comes to life. 

```
require "pry"
require "json"
require 'rest-client'
require 'colorize'

require_relative "moviepod/version"
require_relative "moviepod/cli"
require_relative "moviepod/movie"
require_relative "moviepod/api"
```

### moviepod/lib/moviepod/movie.rb
This file is the movie object, where Ruby will know what the movie is about and what attributes it is composed of. How would I know what? That's where I look into the api website and research what data it can provide to the developer:

```
class MoviePod::Movie 
  
  attr_accessor :id, :title, :overview, :popularity, :release_date
 
  
  @@all = []
  
  def initialize(id,title,overview,popularity,release_date)
    @id = id
    @title = title
    @overview = overview
    @budget = popularity
    @release_date = release_date
    @@all << self
  end
  
  def self.all
    @@all
  end
  
  
end
```

Choosing which one is solely depends on you to decide what information is relevant or not, but for this project I have thought these are the most important one to provide some info about the movie such as its name(title), overview(not to totally spoil the movie ;D), budget, and release date. 

The ID is primarily important which will be explained in the api file below: 



### moviepod/lib/moviepod/api.rb
This file is where the gate opens to the world of data that website provide to you. There are few primary steps: First, you have to know where that data is coming from typically in a form of a link you have found while you were researching the data endpoints and generally they have a link where you can configure, copy, save it in a variable. Look at the code below: 

```
class MoviePod::API 
  
  def api_load
    key = "232880aa87c7a2eb09863ce37088d60a"
    popularity_url = RestClient.get("https://api.themoviedb.org/3/movie/popular?page=1&language=en-US&api_key=#{key}")
    
  
          json_response = JSON.parse(popularity_url)
          json_response["results"].each do |movie_hash|
            
            movie_ID = movie_hash["id"]
            title = movie_hash["title"]
            overview = movie_hash["overview"]
            popularity = movie_hash["popularity"]
            release_date = movie_hash["release_date"]
           
            MoviePod::Movie.new(movie_ID,title,overview,popularity,release_date)
          end
        
  end
  
end
```


First you define a class in which using a local variable of popularity_url(you can name the variable anything that you want) which link to that api and API key is the token that is given to you when you sign up to their website. Sometimes, they are free, sometimes they are not! then you incorporate the api key inside that link to be fully utilize the api. 

in order to pull data out, the JSON format, which is a common standard data structure, needs to be parsed. Rest-Client is one of the ways to parse JSON file in Ruby. Once we required the necessary attributes out of the JSON and saved it in the variable, we can call them by instantiating a new movie object from it and incorporate these attributes that is cooked and ready to be used and manipulated in your CLI below: 

### moviepod/lib/moviepod/cli.rb
This is where all the happenings and business logic take place, where you decide how the users will interact and in what steps. Take a look below:

```
class MoviePod::CLI

  def call
    MoviePod::API.new.api_load
    
    greeting
    trendy_movie_list
    movie_list_menu
  end
  
  def greeting
    puts "Welcome to your movie dashboard".colorize(:green)
    sleep(1.5)
  end
  
  
  def trendy_movie_list
    puts "\nToday's Top popular Movies: ".colorize(:green)
    sleep(1.5)
    MoviePod::Movie.all.uniq.each.with_index(1) do |movie, index|
      puts "#{index}. #{movie.title}".colorize(:blue)
    end
  end
  
  
  def movie_list_menu
    user_input = nil 
    while user_input != "exit"
    sleep(1)
      puts "\nEnter a number from the list to see details about the movie OR 'list' to see the list again, or type 'exit' to exit the program:  ".colorize(:yellow)
      
      user_input = gets.strip.downcase
      
      if user_input.to_i > 0 && user_input.to_i < 21
        sleep(2)
        movie_detail_info(user_input)
      elsif user_input == "list"
        trendy_movie_list
      elsif user_input == "exit"
        goodbye
      else
        puts "invalid character or number entered. Please try again!"
      end
    end
  end
  
  def movie_detail_info(input)
    movie = MoviePod::Movie.all[input.to_i - 1]
    puts "\nMovie#: #{movie.id}".colorize(:blue)
    puts "\nTitle: #{movie.title}".colorize(:blue)
    puts "\nOverview: #{movie.overview}".colorize(:blue)
    puts "\nRelease Date: #{movie.release_date}".colorize(:blue)
  end

  
  
  def goodbye
    sleep(1.5)
    puts "\nThanks for checking moviepod. check us back if you want to check more popular movies".colorize(:green)
  end
  
end

```

The #call method will be completed at the end. It is a simple overview of what methods are executed and in what steps. The very first one comes from the api method we call everytime we loading the cli instead defining in every single method. Chronologically, our app starts with greeting our users, providing them the list, and asking them to either choose a number to see more detail about that specific chosen movie, OR exit the app. Logically, if the user input somethig other than what is provided or allowed in the list, the user will receive an error message guiding them to choose the right number and/or not using anything other than number.



### moviepod/lib/moviepod/version.rb
and the version is nothing but the version of the app you decide. For example, if i add a new features, I would update this to reflect the latest version. Now, in order to execute all of these, take a look below: 


**AND Finally...**

here comes the bin. The bin folder contains nothing but executable file, in this case i named it "run" with no file extension. Inside, I use the famous 'shebang' line to tell the intrepreter that what I use is ruby. Take a look below:
```

#!/usr/bin/env ruby
 
 require_relative "../lib/moviepod.rb"

MoviePod::CLI.new.call

```

The require_relative is a way to execute files that are within my project internal files (api,cli,moviepod inside the lib), and require is used for dependacies in external files such as gemfiles. In this context, I am executing my environment file and also creating a new cli object everytime I execute the file.



**Challenges**

A challenge to start from an empty page sounds overwhelming, but with a little bit of setting a plan of what you want from the api that you feel useful for your own project, how to structure your files to do what each are supposed to do, and testing bugs to find where the logic might be wrong were some of those challenges that I overcame by looking at the videos, walkthroughs, and collaborating and communicating with my team made a huge different as one of the greatest lesson I learnt so far is "as much as software development requires independent and creative thinking, it is also more than essentials that you get involved with your teammates as software engineering field requires teamwork for most part and make it more fun. 

**Future Challenges

Remember about movie ID, one of my object attributes? well, what I have found out is that not all api data endpoints have all the data you need to pull information from. Sometimes, that company or website divided their api endpoint in a way that you need to utilize different api in order to achieve a result you want in your object. 
That movie ID, in the filed of database, is called a primary key, an attribute that is shared among all other data endpoint. My future challenge is, if i need the name of the actors in that film to put it in my own project, because movie ID is a shared data, or a bridge data between these two api, I can now compare the user input result that called a method from the first api to the second api, and then I can able to get information I need from the second api. 

I hope this walkthrough and insights can help you clarify how powerful the api are and design patterns that can be taken to make your app more user friendly. 

Thank you








