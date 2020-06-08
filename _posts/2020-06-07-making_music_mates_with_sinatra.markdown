---
layout: post
title:      "Making Music Mates with Sinatra"
date:       2020-06-07 19:52:08 -0400
permalink:  making_music_mates_with_sinatra
---



Just a few weeks ago, I really didn’t know what I was going to do for my Sinatra project.  Our Flatiron labs leading up to the project were varied enough…and I knew I needed to come up with an idea that didn’t replicate any single one yet utilized relevant structural elements.  Finally, I resorted to thinking about a use case that might appeal to someone I know.  I have a friend who organizes weekly chamber music sessions, so I decided to proceed with him in mind as an end user.  Music Mates was the result.  Music Mates is just the very elemental beginnings of a much more complicated application.  In its simplest form, for the purposes of this project, it’s a directory of musicians that register for membership to the Music Mates community.  Someone signs up and selects from a list all the instruments they play, which will be listed on their public profile, which can be viewed by other members.  A member can also elect to keep their profile private.  You can search for other musicians by instrument from an instrument index, or from instruments listed on a musician’s profile page.  Then you can contact the other members via an email link.

Of course, I initially had bigger ideas for this project – for example, musicians could “level” themselves and others can search for musicians not only by instrument, but at the level they play them at (e.g. Beginner, Intermediate, Advanced, Professional, etc.).  I went as far as perfecting the ActiveRecord searches to do just that, but then realized that the interface would probably go beyond the essentials for the project.  So, I stuck with instruments and privacy as my search constraints.  Eventually, musicians could add audio and video links to their profile, personal statements, preferred repertoire, availability, etc.  All these functions are now on my list of stretch goals for future implementation.

Music Mates is based on 3 models: User, UserInstrument, and Instrument.

Originally, UserInstrument had an additional attribute of “level,” but I removed it for the time being to keep my database limited to what I’ve implemented for this project.  I decided to use checkboxes for users to select the instruments they play (a standard prescribed list for now), and radio buttons to select either a public or private profile.

Checkboxes actually handle their own updating in Sinatra with ActiveRecord, and using them posed an interesting challenge and a bug that needed to be addressed.  I used a complex form in the view for my Edit Profile page and rendered the checkboxes with the following code:

```
<% @instruments.each do |instrument| %>
                <div class="form-check form-check-inline">
                    <input class="form-check-input" type="checkbox" name="user[instrument_ids][]" value="<%= instrument.id %>" id="<%= instrument.id %>" <%= 'checked' if @user.instrument_ids.include?(instrument.id) %>>
                    <label class="form-check-label" for="<%= instrument.id %>">
                        <%= instrument.name %>
                    </label>
                </div>   
            <% end %> 

```

The selected checkboxes will return within the params[:user] as an instrument_ids array that will update in the user_instrument table, taking care of any deletions and additions automatically.  Sweet!  EXCEPT, what if a user deselects all their instruments?  Then NO array is sent at all, and that causes a bug.  With no instrument_id array, nothing in the user_instrument table gets updated and therefore, all the previously selected instruments remain in the database and will continue to load in the profile.  This little bit of extra code at the beginning of the patch route takes care of this issue:

```
 if !params[:user].keys.include?("instrument_ids")
      params[:user][:instrument_ids] = []
    end
```

Essentially, if the params hash does not include a key of “instrument_ids,” then it signifies that all instruments were unchecked and therefore no array was passed.  In this case, a key of instrument_ids pointing to the value of an empty array is added to the params[:user] hash, and now all the existing instruments for the user will be eliminated from the database with the mass params[:user] update.

The specs for the Sinatra project suggest that CRUD operations should be applied to the belongs_to resource.  But the nature of this project revolves around user selection, not so much user product, so permission was granted to let the checkboxes take care of the UserInstrument CRUD, and just make sure I demonstrate all explicit CRUD with my user resource, which I did.

One feature that I elected to implement in this initial version of Music Mates is public vs. private profiles.  I set a default parameter to “public” for signup, since having a public profile and wanting to be found is pretty much the purpose of Music Mates!  The key line of ActiveRecord code I use to search for public profiles by instrument is:

```
User.where(visibility: "public").joins(:user_instruments).where(user_instruments: {instrument_id: params[:id]})

```
This code only considers public users, then searches by specified instrument.  I eventually sort the array by user’s last name.  I suppose I could have done it in the query, but the line was long enough already!

Extra little coding macros I used in my user model included, `dependent: :destroy` and validators for presence and uniqueness of relevant attributes and to ensure the integrity of the data stored in the database:


```
class User < ActiveRecord::Base
    has_secure_password

    has_many :user_instruments, dependent: :destroy
    has_many :instruments, through: :user_instruments

    validates :first_name, :last_name, :email, :visibility, presence: true

    validates :email, uniqueness: true

end

```
Since I am allowed a user to delete their own account, the `dependent: :destroy` macro ensures that any entries in the user_instrument table pertaining to the user are also deleted from the database.

In addition to these macros, I made sure that my routes made use of helper methods that served to check for login status, current_user and would issue Flash error messages should a user try to alter another user’s profile through the address bar.  In addition, other user profile pages eliminate the edit and delete buttons.

I also utilized ActiveRecord’s error method, which taps into the validation macros set up in the user model.  Upon failure when triggered by .save or .update, the error method informs the user why the signup or update failed:

`Account creation failed:  #{@user.errors.full_messages.to_sentence}.`

Such code would generate a message like: “Account creation failed: Last name can't be blank.”

Finally, I decided to use the Sinatra project to familiarize myself with Bootstrap 4.5.  It was extremely easy to use and allowed me to create a clean interface, particularly for my Edit Profile view, the most complicated one, and a navigation bar whose content changes once a user logs in.

I really enjoyed working on the Sinatra project and learning to deal with everything from working with a database, creating models, and connecting everything to a secure user experience.  Forcing myself to pare down my initial vision really helped me to get comfortable with the general mechanics of creating a web app.  It’s possible to take pride in something simple, yet functional.  It’s the first step in creating more complex web applications that can deliver whatever one’s mind can conceive of!

