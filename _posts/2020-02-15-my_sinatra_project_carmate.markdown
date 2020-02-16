---
layout: post
title:      "My Sinatra Project: Carmate"
date:       2020-02-15 19:52:51 -0500
permalink:  my_sinatra_project_carmate
---


Sinatra project is one of my  favorite projects that I could finally  able to utilize Ruby functionality to create something visually more appealing and more functional. Carmate, as it may sound, is about a hypothetical interface of a fictional car dealership specifically served for employees who can create their own account, which means signing up with their full name, email, and creating a username and password, and then they can able to create a car with its details: 


![](https://i.imgur.com/0zu1AGU.png)

But before getting into the details, you either can clone this [repo](https://github.com/pjavadi84/carmate) into your own framework and start testing this out, or follow my blog to show some highlights. So let's get coding:

So here is a general file structure of the Corneal, a Sinatra tool for building quick Sinatra-based application. It saves you time on creating initial required gems and data structures from scratch:

![](https://i.imgur.com/KywLsfZ.png)

To break this down, Corneal and Sinatra provide a great framework to get started. As you see, I have my models, controllers, and views being setup. The very first thing you should think about is where you want to start your project first. I prefer to build the building blocks first and that starts with the database. 

When you are have installed your corneal gem, you can create your database by typing into your command line the following code. The NAME should be the name of your object; in my case, it is USER and CARS. 

`corneal scaffold NAME`

What the code above does is it will create a file template along with database/migration files. Once everything is being created, you can create your migration attributes. For instance, my user has a first_name, last_name, username, and password. 

**DATABASE/MIGRATION FILES**

My user database looks like this:

![](https://i.imgur.com/DduXLU7.png)

This password_digest comes from bcrypt gem defined in your gemfile. What it does is it basically, encrypt the password that your end-user will input to be persisted into your database. It is a security feature that creates that extra level of security from hackers and intruders to retrieve those password. For instance, it change the password from "john123" to something like this "A$OUOdkljlkw0)(#@)9sldkjf2)(#@)$(R8ews...." so this is what the hacker might be able to see but it cannot be copied/pasted and can't be guessed. The action of converting the password to these text is called "salting".

my user database looks like this:

![](https://i.imgur.com/wmnOawv.png)

You might ask, what the "user_id" is doing in the car database. When we create the user and car objects, we have to associate them. There should be something related between these two model. We will cover the model section in a bit, but literally providing user_id in the car is column pertaining to the foreign key. Because a USER can have MANY CARS, and a CAR belongs to a USER. So, the foreign key will always resides in the "many" side of the relationship to be unique instances. 

**MODELS**:

My user model looks like this:
![](https://i.imgur.com/31pVWVO.png?1)

and my car model:
![](https://i.imgur.com/CM1IZb3.png​)

Here we define association between these two models, but since we want to implement security features on the users, because they have password, we add  "has_secure_password" to utilize bcrypt and "validates_uniqueness_of :username" to identify the 'username' as a unique identifier of a user. It can be email, firstname, basically whatever you want but industry standards are based on username in the world of web usually. 

Once all setup, we use "rake db:migrate" . This should create your schema, a blueprint of your database structure and its contents:

![](https://i.imgur.com/qp403Ii.png)



We are done with setting up our database, associations, and models. Now, we can move on to setting up our controllers and views:

We create unique controllers for each objects that we try to create. It create seperation of concerns, organizing business logic of our app into dinstinct containers that are designed to handle its specific interactions. In our case, we have a user controller that handles user interactions, signup, logins, logout, in our app:

**User Controller:**

```
class UsersController < ApplicationController


  # GET: /users/new

  get "/signup" do
    if !logged_in?
      erb :"/users/new.html", locals: {message: "can't logged in. Please create an account"}
    else
      redirect to "/cars"
    end
  end

  # POST: /users
  post "/users" do
    if params[:first_name].empty? || params[:last_name].empty? || params[:username].empty? || params[:password].empty?
      flash[:message]="field can't be empty! Please fill out all the information and try again."
      redirect to '/signup'
    else
      if User.find_by(username: params[:username], email: params[:email], first_name: params[:first_name], last_name: params[:last_name])
          flash[:message]="user profile already exist in our database. please try another email."
          redirect to '/signup'
      else
          @user = User.new(:first_name =>params[:first_name], :last_name => params[:last_name], :username => params[:username], :password => params[:password])
          @user.save
          session[:user_id] = @user.id
          redirect to "users/#{@user.id}"
      end
    end
  end

  get '/login' do
    if !logged_in?
      erb :'users/login.html'
    else
      flash[:login_success] = "login successful!"
      redirect to '/cars/show.html'
    end
  end

  post '/login' do
    @user = User.find_by(:username => params[:username])
    if @user && @user.authenticate(params[:password])
      session[:user_id] = @user.id
      redirect to "users/#{@user.id}"
    else
      flash[:message]="login credential failed. Please try again!"
      redirect to "/login"
    end
  end

  # GET: /users/5
  get "/users/:id" do
    if logged_in?
      @user = User.find_by(id: params[:id])
      erb :"/users/show.html"
    else
      redirect to "/login"
    end
  end
  
  get '/logout' do
    if logged_in?
      session.destroy
      redirect to '/login'
    else
      redirect to '/'
    end
  end

end

```

and, our** Car Controller:**

```
class CarsController < ApplicationController

  # GET: /cars
  get "/cars" do
    if logged_in?
      @cars = Car.all
      erb :"/cars/cars.html"
    else
      redirect to '/login'
    end
  end

  # GET: /cars/new
    get "/cars/new" do
    if logged_in?
      erb :"/cars/new.html"
    else
      redirect to '/login'
    end
  end

  # POST: /cars
  post "/cars" do
    if logged_in?
      if params[:make] == "" || params[:model] == "" || params[:color] == "" || params[:car_type] == "" || params[:price] == ""
        redirect "/car"
      else
        @car = Car.create(
          make: params[:make],
          model: params[:model],
          color: params[:color],
          car_type: params[:car_type],
          price: params[:price],
          user_id: current_user.id
          )
          
          redirect to "/cars/#{@car.id}"  
        end
      else
      redirect to '/login'
    end
  end

  # GET: /cars/5
  get "/cars/:id" do
    if logged_in?
      @car = Car.find(params[:id])
      erb :"/cars/show.html"
    else
      redirect to "/login"
    end
  end

  # GET: /cars/5/edit
  get "/cars/:id/edit" do
    @car = Car.find(params[:id])
    if logged_in?
      if edit_authorized?(@car)
        erb :"/cars/edit.html"
      else
        redirect to "users/#{current_user.id}"
      end
    else
      redirect to "/login"
    end
  end

  # PATCH: /cars/5
  patch "/cars/:id" do
    @car = Car.find(params[:id])
    if logged_in?
      if edit_authorized?(@car)
        @car.update(make: params[:make],model: params[:model], color: params[:color],car_type: params[:car_type],price: params[:price])
        redirect to "/cars/#{@car.id}"
      else
        redirect to "users/#{current_user.id}"
      end
    else
      redirect to '/login'
    end
  end

  # DELETE: /cars/5/delete
  delete "/cars/:id" do
    @car = Car.find(params[:id])
    if logged_in?
      if edit_authorized?(@car)
        @car.destroy
        redirect to '/cars'
      else
        redirect to "users/#{current_user.id}"
      end
    else
      redirect to '/login'
    end
  end

end

```

We can create a central controller hub called application controller that modularize all our controllers into one and provide an interface to communicate with our environment file:

**Application Controller:**

```
require './config/environment'
require 'sinatra/flash'

class ApplicationController < Sinatra::Base

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions
    set :session_secret, "carmate_secret"
    register Sinatra::Flash
  end

  get "/" do
    if logged_in?
      redirect to "/users/#{current_user.id}"
    else
      erb :welcome
    end
  end

  helpers do

    def logged_in?
      !!current_user
    end

    def current_user
      @current_user ||= User.find_by(id: session[:user_id]) if session[:user_id] #(memoization)
    end

    def edit_authorized?(car)
      car.user == current_user
    end
  end

end
```

If you look above, under configure we setup our user session, that is used to enforce session through our controllers whenever they need to be activated. For example, if we need our user controller to verify if the user is in logged_in, only by having a session we can proceed to move to the next step in our app; otherwise, without it anyone can access other's account and manipulate their data! imagine, Facebook and Instagram didn't have this session requirement. I could have edited random people's information without any problem. Sweet! :D 

We also define our helper methods. Our helper method, as its name implies, are created to help us call them whenever we need them without explicitly define what we need in each block of code literally everytime. Without them, the "code smell" becomes a serious thing. So use them to your advantage. In this particular case, we have a logged_in helper that ensure if we need to edit our car listing, it is the user who created it initially can access this information so we "callback" our logged_in helper to come to our aid. 

As previously discussed, when we use "corneal scaffold NAME", it will also create the descriptive views. These views are html files that our users see and are controlled by our controller CRUD actions. For instance, in our user controller we need to show the user the signup page. We use "GET" http request to render an erb view for signup in our user's view. Then when we shogun and test and navigate to signup page, that's the signup erb file that is being rendered and shown to us as user. Once we input information, those information needs to be stored in params and being send to POST request in our controller. In another words, we imagine whatever the user enters in the form are passanger, the params is the bus that moves between GET and POST requests. 


MVC framework create a very seamless transition between these http requests and makes our jobs as developer easy and deploying apps much faster. 

If you are interested to see each view files for users and cars, please visit my [github](https://github.com/pjavadi84/carmate/tree/master/carmate/app) page and navigate to the views and see how each page is created when GET or POST requests are being implemented.
