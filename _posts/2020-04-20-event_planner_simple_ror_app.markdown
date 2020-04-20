---
layout: post
title:      "Event Planner: Simple RoR app"
date:       2020-04-20 05:02:25 +0000
permalink:  event_planner_simple_ror_app
---


# EventPlanner: My simple RubyonRails(RoR) Project

Thus far, one of the most challenging project I have encountered thus far was my RubyonRail porfolio project. I should confess half of the challenge was about the idea that I should have to brainstorm and halfway through my first week of project, everything failed because of the lack of possible imagination, but a great lesson that I have learnt from it are: For a slow learner like me, I feel the easier way to come up with coming up with an idea is through many apps that we use on a daily basis. 

Take Facebook or Instagram for instance; I didn't thought about creating that much sophistication, but i thought about a small piece of milion of other functionalities and interactions that these giant apps are operating on. I took facebook event planner as an example, and within it, one of the basic functionalities is COMMENTS. But, then, i thought about how can i come up with a comment. another object or entity within this app: a USER. Think about the name of the app: an event planning! Usually, an event is an entity that needs to be physically setup somewhere so people can come in there and hangout. Without a PLACE(property model), there will be no event(event model), and without the event, there is nothing that people can interact with, such COMMENT(comment model: 

So, let's start programming:

First thing first, we need an skeleton. Probably the second most important thing you need to thing about after the story process lifecyle of your app, is your models about these entities, and where all of these model are made of and placed in the app. So first step is always: setting up your new project on your project terminal directory:

```
rails new eventplanner
```
This will generate your basic project framework and file structure template for your rails application. Then, you will create your models: 

User, Property, Event, and Comment

Next, I created my migration(database) files:

```
rails g migration CreateUsers
rails g migration CreateProperties
rails g migration CreateEvents
rails g migration CreateComments


```

Once this done, you might think you have fill up your migration tables with attributes, but before doing all of that, let’s think about how models are associated with each other. I know that everything is start with my user, because the very first thing this app need is that someone needs to be “signed up” to have their specific account and can be able to perform their own activities and modify and query data from it. So, this is how my models are set up: 

```
User:
class User < ApplicationRecord
    has_many :properties
    has_many :comments
    has_many :events, through: :comments
    has_secure_password
end


Property:
class Property < ApplicationRecord
    belongs_to :user 
    has_many :events
    

Event:
class Event < ApplicationRecord
    belongs_to :property
    has_many :comments
    has_many :users, through: :comments

Comment:
class Comment < ApplicationRecord
    belongs_to :user
    belongs_to :event
end




```
So based on my associations, comment is the joint table between my users and events that has been created from the property. So, first I create the USER, then Property, and from Property I can create my Events. Once event is being created, user can comment on those events because user can access many events and each event can host many users to comment on. 

Once all of these set up, I use:



```
Rake db:migrate 
```

So, this is will make my tables ready to save users, properties, events, and comments. Then, we create controllers for each of these: 


https://imgur.com/nKAG8fR

Omniauth: 

I have used Github as my third party user authentication provider

I have created indexes for my properties and events which shows all the properties the user creates and because events are nested within properties, you can go directly into that property and see all the events that are hosted in there. For example, take a look at these two indexes: 

https://imgur.com/HHM90ZX


And the events are within the show details of each property:

https://imgur.com/tYe8v5U

And this is where user can see the specific event and see comments. Only the users who are signed in can delete their own comments, and can’t delete other user’s comment:

 https://imgur.com/IbeBVy2

So this is in nutshell, where my comments are being utilized where user can “destroy” a comment and the comment can be shown on the event show page because comment is the joint table. 

