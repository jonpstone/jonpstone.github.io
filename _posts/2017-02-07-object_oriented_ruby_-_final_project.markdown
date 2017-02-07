---
layout: post
title:  "Object Oriented Ruby - Final Project #"
date:   2017-02-07 20:40:42 +0000
---

**Background**

So, this is my first blog post. I've actually been with Flatiron's online Learn.co campus for over six months now, there's been some personal stuff (nothing bad), work, Christmas, family and a not so healthy dose of procrastination getting in the way of progress. I also made the unwise choice of pushing on with the curriculum instead of stopping and taking the time to complete projects. While this is doable, I really would not recommend it. Going back from Rails, where I am now, to raw Ruby was, at times, frustrating and I also have to now switch back to Sinatra for that module's project, which I haven't touched in a good while. However here we are, I think I'm done with OO Ruby, with a bit of help from the resources made availiable on the lesson page, Corinna and some Googling, I got there; so below is a little on how I did it. Below is a link to the GitHub repo.

https://github.com/jonpstone/project-cli-dogparks

**Getting Started**

One of the ways I got started here was by watching the great video done by Mr. Flombaum! I watched it all the way through and then, while I took to my text editor, kept my finger on the play button in case I got stuck. I decided on creating a gem to provide a user with data on local area dog parks in the city of Plano, TX. This basically concerned one park in Plano itself and then two others in Frisco and North Dallas, each dog park had a seperate website, from which I would have to scrape data. Aping Avi, I got started by using bundler to flesh out the file tree.

Once the completed I made a file in the `./bin` folder and adjusted the persmissions to make it executable. This was not a ruby file and required the 'shebang' on line 1. This would be the file which would launch my gem.

*./bin/dog-parks*
```
#!/usr/bin/env ruby

require "bundler/setup"
require "project_cli_dogparks"

ProjectCliDogparks::CLI.new.call
```

I then created a `cli.rb` and a `park.rb` class, both located in the `./lib/project_cli_dogparks` directory. The `project_cli_dogparks.rb` file also within `lib`, features an empty module (created by bundler), this was used, as the project progressed, to house the various requirements for the gem and any objects to interact; an environment.

*./lib/projectclidogparks.rb*
```
require "open-uri"
require "nokogiri"
require "pry"
require_relative "./project_cli_dogparks/version"
require_relative "./project_cli_dogparks/park"
require_relative "./project_cli_dogparks/cli"

module ProjectCliDogparks
end
```

**The cli.rb class**

This was the cogs and chains of the gem and would guide the user through the program. As you can see in `./bin/dog-parks`, the executable calls the `#call` method within the `cli.rb` class. This in turn then calls two methods within the class, first `#list_parks` and then `#choice`. We'll take a look at the former below.

*./lib/projectclidogparks/cli*
```
  def list_parks
	
    puts "Welcome to the Plano area Dog Parks gem!"
    @park = ProjectCliDogparks::Park.all
    @park.each_with_index do |park, i|
      puts "#{i+1}. #{park.name} - #{park.city}"
    end
		
  end
```

To kick things off for the user, they're greated with a simple string, which prints a welcome message to the terminal. I created and instance variable `@park` and assigned it the value of a new instance of `park.rb`. An 'each' loop cycles through that instance and returns an ordered list produced by a concatenated string of attritubes. This is taken from scraped data in `park.rb`, it includes the name of the dog park and the city it is located in. At this point the `#choice` method has been called, as evidenced by the final line printed in the terminal below. The user is presented with the following in their terminal.

*TERMINAL*
```
Welcome to the Plano area Dog Parks gem!
1. Dog Park at Jack Carter Park - Plano, TX
2. Ruff Range Dog Park - Frisco, TX
3. NorthBark Dog Park - Dallas, TX
Please make a selection or type 'exit' to leave or 'menu' to return to the list.
```

It is then up to the user to decide which park they would like to know more about. When the selection is made the `#choice` method really gets put into action. In this method (see below code snippet) I set a variable called `input` equal to `nil`. After which, a 'while loop', which states that while the user `input` is greater than 0 and less then 4, begins. It is here that `input` is set to `gets.strip.downcase` in order to retrieve the relevant data for the user's selection. The new value of `input` is assessed further by more conditional logic in the form of an 'if statement'. This will discern whether the user wants to view item 1, 2 or 3, or whether they simply wish to return to the ordered list or exit the program. This method will also identify invalid choices and prompt the user to make another selection. If the user makes a choice within the indexed options, the method will pull data from the `@park` instance, now stored in the `user_park` variable. This block then returns a set of attributes for user_park within a heredoc.

*./lib/projectclidogparks/cli*
```
  def choice
	
    input = nil

    while input != "exit"
      puts "Please make a selection or type 'exit' to leave or 'menu' to return to the list."
      input = gets.strip.downcase

      if input.to_i > 0 && input.to_i <= 3
        user_park = @park[input.to_i-1]
        puts <<~EOF

          #{user_park.name}\n
          #{user_park.location}\n
          #{user_park.description}\n
          #{user_park.url}

        EOF
      elsif input == "menu"
        list_parks
      elsif input == "exit"
        so_long
      else
        puts "Not a vaild choice."
      end
    end
		
  end
```

In the case that a user wishes to exit, the `#so_long` method is called, which simply `puts "Have fun walking your dog(s)!"`.

**The park.rb class**

So, down to the where the data comes from. It's time for `park.rb` to do it's stuff! As you may recall, when `#list_parks` is called by `cli.rb`'s `#call` method, it requires data from a class method called `#all` in `park.rb`. This method in turn calls another class method called `#scrape_parks`. The latter method houses an array called `parks`, this array is populated by a scraper method that I will talk about next. Below is `#all` and `#scrape_parks`.

*./lib/projectclidogparks/park.rb*
```
  def self.all
	
    # Scrape Plano area dog park .gov websites for data.
    self.scrape_parks
		
  end

  def self.scrape_parks
	
    parks = []

    parks << self.scrape_plano
    parks << self.scrape_frisco
    parks << self.scrape_dallas

    parks
		
  end
```

For the sake of brevity I'll just share the scraping method for 'Ruff Range', which is the dog park located in Frisco, TX. Basically all the scraper methods are more or less the same (with the exception of some data that I irritatingly had to hard code, especially in the Plano method).

*./lib/projectclidogparks/park.rb*
```
  def self.scrape_frisco
	
    doc = Nokogiri::HTML(open("http://www.ci.frisco.tx.us/Facilities/Facility/Details/Ruff-Range-Dog-Park-54"))

    park = self.new

    park.name = doc.search('.section.address h4[1]').text

    location_street = doc.search('p.adr span.street-address').text
    location_city_state = doc.search('p.adr span.locality').text.gsub(/(?<=[a-z])(?=[A-Z])/, ', ')
    location_zip = doc.search('span.postal-code').text

    park.location = "#{location_street} " + "#{location_city_state} " + "#{location_zip}"

    park.city = "#{location_city_state}"

    park.description = doc.search('#moduleContent #FacilitiesContent div div div font').text.strip

    park.url = "http://www.ci.frisco.tx.us/Facilities/Facility/Details/Ruff-Range-Dog-Park-54"
		
    park
		
  end
```

So I began by creating a variable call `doc` to which I assigned the value of a url to be scraped by 'nokogiri'. I would test for the data being collected using 'pry' in the terminal. I had to require both gems in my environment - './lib/projectclidogparks' and gemspec file to utilize these tools. Using the terminal I ran the gem until it hit the `binding.pry` line, which at this time along with the `doc` variable was the only code inside the method. Using Chrome dev tools I was able to identify the data that needed to be scraped. Some data, like the text for the park description for Plano, was not placed inside of an element and had to be hard coded. For the most part, however, this was not an issue and the scraping was quite straightforward. The only other exception was the `.location` attribute, when I simply tried scraping the `.street-address` element. Even when applying `.strip` in a method chain it looked kinda messy, it also returned `FriscoTX` rather the `Frisco, TX`, which I required for the `.city` attribute. So I improvised a block of code scraping different bits and peices that were then concatenated together as a value for the `.location` attribute.

I won't give an example of the output below, feel free to clone and test it out for yourself! Needless to say, as per #choice, the user is provided with expanded information regarding the specific street address, a description of the park and a url for the park's website. 

Well that just about wraps it up, hope I was clear and, more or less, concise! Peace all and I'll see you next time.
