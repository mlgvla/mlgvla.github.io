---
layout: post
title:      "Benning Violins - My CLI Project"
date:       2020-04-15 22:40:41 -0400
permalink:  benning_violins_-_my_cli_project
---


This was really quite an adventure!  I went through several different project ideas, when it finally occured to me to code a project concerning web content I knew something about...string instruments.  I figured that would help me figure out the most important information to scrape from a website.  So, I picked a  string shop that sells fine violins, violas, cellos, bows, plus instruments made by the Benning family.  I have a lot of friends that deal in musical instruments, so I had them in mind as an end user.

I divided my project into 4 classes:  CLI, Inventory, Instrument, and Scraper.  First, I worked out my scrapes using Scraper Checker on Repl.it.  There were several challenges here.  One of my menus had nested selections, so I had to scrape two separate menus and then combine them to make the main Inventory menu in my application. I had severall issues with the data in my scrapes.  One was dealing with HTML non-breaking spaces.  The other dealt with label names in `<strong>` tags being included in my data scrapes.  For example, "**Instrument Maker:**  Douglas Cox" After poking around the Internet, I found a simple solution using Nokogiri. to deal with the non-breaking space.  I also had to come up with a Regex to eliminate the label, leaving me with the pure data I wanted to store.  However, the labels in the HTML did help to match the data with the attributes in the Instrument Class.

```
nbsp = Nokogiri::HTML("&nbsp;").text #convert HTML nbsp to text nbsp
      new_detail = detail.text.sub(/^[^:]+:\s*/, "") #delete label
      final_detail = new_detail.sub(nbsp, "") #delete text nbsp
```

Anyway, once my scrapes were cleaned up, then it was time to code things up.  I started with my CLI class "writing the code I wished I had," and hard coding data just to get the mechanics working.  When I first set up Visual Studio Code, I had some problems getting Pry to working properly.  Essentially, I had my Inventory and Instrument classes call the Scraper class, and I had the Scraper Class create Inventory and Instrument objects and plop the scraped data into them.  That way, my CLI could focus on user interface.  I waffled on whether or not I would have my Scraper class return Instrument details in a hash and use #send in the Instrument class to assign them to attributes.  I decided to assign the attributes in the Scraper Class for now.  But I will probably refactor this using the return hash/#send as another possibilty.

My project goes two levels deep.  User selects an inventory, and then that inventory is displayed.  Then the user can select a particular instrument and see the details about that instrument, including a link to the website's details page in case the user wanted to view an image.  Another annoying problem was the terminal display of the "Description" detail, a huge text dump.  The text went to the edge and wrapped in the middle of words.  So, I went to Ruby Gems and found "word_wrap," which can extend the String class so appending .wrap to a string will limit the line to 80 characters and wrap on the white space.  It took a bit of figuring out how to "require" the gem.  Turns out it takes 2 requires in your environment file to get it working:

```
require 'word_wrap'
require 'word_wrap/core_ext'
```

Reading the "issues" on GitHub, cleared that up for me.

Of course, I did spend a lot of time testing my program.  It seemed to be unbreakable...until I attempted to see details on a particular violin, which the website failed to have a webpage for.  The dreaded "404 - Page Not Found" error and my program terminated. I went over to the website, and sure enough, the web page for that link was miising. What to do?  I had to  include all instruments in the inventory.  So I looked at the error on my terminal and saw that it was raised by OpenURI:HTTPError when the page was not found.  Time to review error handling, which was covered in a previous module.  I poked around the Internet and looked as several articles that illustrated how to rescue your program from a specific error.  I played around with some code in Repl.it to gain further understanding of how to manage good and bad webpage links and came up with a block of code that I put into my scrape_instruments_details method:

```
 site = BASE_PATH + instrument.url

    begin
    opened_site = open(site)
    
    rescue OpenURI::HTTPError
      puts "Page Not Found.  Details Not Avaiable for this Selection."
      return
    end
```

This code handles the missing web page, explains to the user the issue, returns out of the method, leaving NIL values in the details for that violin.  My CLI class displays NIL detail values as "N/A" (Not Available).  If no error is generated, the method runs normally and fills in attributes with details.



The wonderful team at "Ask a Question" has to essentially "shun" you duriing your CLI Project.  It's tough love, to be sure, but you learn SO MUCH from having to solve all these issues on your own.  The actual coding of my project was the easy part - all our labs and video demonstrations of project creation really help with that.  But  I had to figure out how to fix my Pry issue (2 installs seemed to correct that), dealing with HTML non-breaking spaces (thank you Stack Overflow!), and getting my environment file set up, I learned that you need to fill out your Gemspec or your "irb" will not work and spew out the worst possible mess.  I feel like I really understand how all of this is organized now.  All of that gave me the confidence to try and figure out how to rescue my program from an error that was caused by an external factor.

Yes, there is more I could do to my project to improve the interface, but what I have now seems to work OK.  The user can chose and inventory, look at it, and pick an specific item to see more details.  Then they can pick a new inventory and essentailly rinse and repeat until they've had enough and type "exit".  There comes a point where you have to say that it's OK to move on.  Just make a refactor wish list and try to tinker with it more in those odd moments or when you want to develop a new skill.

This was really fun, once I got past the initial shock of trying to get my environment to work and the files to work together properly.  I'm really looking forward to all the other independent projects and the various challenges they will undoubtedly pose!!!

