---
layout: post
title:  "Sinatra: Fly me to the..."
date:   2017-04-23 04:48:28 -0400
---


## Concept
Air War!!! I decided that, although I haven't played any video games for a while due to work and school (much to my Wife's delight also, as she gets to watch her shows in the living room and has done for nearly a year...) that I would design a Sinatra web app for Mass Multiplayer Online (MMO) flight simulator communities to keep track of their team's progess and stats. Well, what do you call a team that fights in the sky, a Squadron... so **COMBAT SQUADRON** was thus conceived. This was a fun idea because I'm something of an armchair historian (the Seed data I assembled was based on World War II aircraft and historical pilots) and it reaches back to my own time served in the British Forces.

## Objects and Associations
So I needed to think about what objects might be present in such an app and what kind of relationships they might have with one another. it's obvious that as in any squadron, you'll have *Pilots!* *Pilots* in 'COMBAT SQUADRON' would be filling the role of the *user* object that another app might have for someone to interact with, remove, add and alter data through an account. I figured that a *Pilot* may well fly more than one *Plane*; almost certainly in his career (and definitely in a video game). Naturally *Planes* would be another type of object... you're not much of an Air Force if you don't have any *Planes*. So, looks like *Pilots* would have many *Planes*, but I also reckoned that *Planes* could have many *Pilots*. Obviously the *Plane* object more accurately represents a **type** rather than a single entity of that Plane, à la the P-51 Mustang is a type, whereas **GUNFIGHTER** is the name of one such P-51. My intention is for the *Plane* object to represent the former. There's no Air Force without *Planes* and it wouldn't be much of an Air **Force** if those *Planes* weren't armed! The third and final object would be the *Weapon*, multiple *Weapons* can be assigned to a *Plane* and mulitple *Planes* can have one *Weapon* type assigned to them. Ultimately I ended up with a Many-to-many relationship between *Pilots* and *Planes* and the same relationship between *Planes* and *Weapons*. There is was relationship required between *Pilots* and *Weapons*, not at least that seemed to serve any purpose.

### Visualizing and Organizing
How did I want the data displayed and where? I decided on a show all page for each class of object. I thought that tables were the best way of tidying up an unordered list, although I'll probably just use divs for my next project. These would be essentially `Stuff.all` style HTML pages, which I called 'Squadron Roster' for the *Pilots*, 'Aircraft Hangar' for the *Planes* and 'Station Armoury' for the *Weapons*. I also naturally wanted to be able to display one single selected object from each class and to be able to provide views for making, altering and removing data and objects as per CRUD. The end result is that I had three different Controllers that look rather similar, though I don't think that this, in of itself is perhaps all that unusual. There were some notable differences though, for example using the session hash was crucial for the *Pilots* controller and for Logging in/out,  creating *Pilots* and even Deleting; where I eventually figured out that I needed to put a `session.clear` line after where the `@pilot.delete` method was used.

```      
if @pilot.id == current_user.id
	@pilot.delete
end
session.clear
redirect to '/login'
```

In terms of file organization I followed much of what I had seen beforehand in the Sinatra Labs and also reused much of the non **app** or **db** files from my Fwitter assignment, though of course I had to edit some things like `config.ru`. Inside of **app**, I used the normal MVC sub-directory set up I had gotten used during labs.

## Helpers
I made use of several helpers in this app that will be familiar to most, I placed them in the parent controller `application_controller.rb`.

```
  helpers do
    def logged_in?
      !!session[:id]
    end

    def current_user
      Pilot.find(session[:id])
    end

    def username
      session[:username]
    end
  end
```

The first is farily self explanatory, in that the method is asking whether or not the *Pilot* is not not logged in! I did read the logic behind being not not logged in on Stack Overflow, but why the precise reasoning escapes me at this time. Next up is a method stating who the current user is by way of the :id in the sessions hash. This was useful for validating who could do what to what and where. Lastly the username method is purely there to provide a reminder in `layout.erb` as to who is logged in. This post isn't as long as my 

## Frontend Fun
One thing I enjoyed was going back to some of those frontend lessons I took at the beginning and digging up some of that knowledge. Although I have to say, much of what I used in my final code is stuff I found when googling. I'm sure things like the `<style>` tag were in the opening lessons of the Curriculum. However, the truth is I found a lot of the code I was looking for at places like [Stack Overflow](http://stackoverflow.com/) and [W3Schools](https://www.w3schools.com/). Those lessons were great, but just so long ago and the above resources were just quick to get the guiding reference I needed. This included some CSS used internally for the `layout.rb` file where I used the `<% yield %>` for other pages to display data. I played around with rack-flash and displayed warning messages in red when boxes were left unfilled or a *Pilot* tried to do something without the right permissions, like delete a plane that they didn't fly. I used checkboxes, as in the NYC lab, extensively in the create and edit views. I really wanted to find a way for existing relationships between objects to be relected in the edit views, such that some boxes would already checked to represent a *Pilot* that already has one more *Planes* when you enter the edit view for that *Pilot*. I'm sure the instructor that reviews my project might be able to tell me. Until then, please feel free to check out [**COMBAT SQUADRON**](https://github.com/jonpstone/portfolio-project-sinatra-combat-squadron) and happy trails!

![](http://orig09.deviantart.net/3fa5/f/2015/171/a/f/2_by_roen911-d8y27xx.jpg)
