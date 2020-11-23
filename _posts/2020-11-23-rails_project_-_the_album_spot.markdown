---
layout: post
title:      "Rails Project - The Album Spot"
date:       2020-11-23 23:16:42 +0000
permalink:  rails_project_-_the_album_spot
---




 ![](https://i.imgur.com/9FpabzY.png)

For my Rails project, The Album Spot, I wanted to work with an API to gain experience working with an external database and mapping it into my application’s database.  Spotify has a terrific API that gives one access to an extensive database of artists, albums, music, and general information.  I decided to write an app that would allow the user to search the Spotify database of artists, view the artist’s albums, and add albums to their personal collection in the app.  Plus, the user can listen to the albums in their entirety, provided they have a Spotify account.  Finally, a user can write reviews and rate their albums and read other user’s reviews of albums.  I also wanted this app to be visual, as well, so I used album art throughout the app to make it feel more like a real album collection.
I utilized several Ruby gems in writing this app.  I used Omniauth for the option to log in through Github. RSpotify is a gem that acts as a Ruby wrapper to facilitate accessing data from the Spotify API.  It necessitates obtaining a Client ID and Client Secret, since most requests to the API need to be authorized.  I put this information into an initializer file using ENV variables for the Client ID and Client Secret
I also made extensive use of Bootstrap for styling.  Using the Bootstrap-Form gem helped to keep my forms uniform in appearance to the rest of the app.  I also used some star icons from Font Awesome for my star rating system.
The Album Spot uses five models:  User, Album, UserAlbum, Review, and Artist.  UserAlbum acts as the join table:

![](https://i.imgur.com/zGZSJCX.png)

Note that a UserAlbum has one Review and a Review belongs to a user album.  This association was used throughout the app!!!!

The first major challenge was how to map the Spotify Album data to the Album table in my local database.  The Spotify API has extensive documentation describing their various objects (Artist, Album, Track, etc.), so you know exactly what information is coming in and the exact datatype of each attribute.  In my Album model, I created a class method self.create_album_from_spotify(spotify_album), which takes as an argument a Spotify Album object retrieved from the Spotify API.  It checks if the album or related artist already exists in the app’s database, and if not, creates one or the other or both, storing only the data that the app will be using including the album art and unique Spotify album identifier, which is important for the Spotify widget I used to embed the actual album tracks into the album show page:

```
def self.create_album_from_spotify(spotify_album)
    
        artist = Artist.find_or_create_by(name: spotify_album.artists[0].name) 

        album =  Album.find_or_create_by(spotify_id: spotify_album.id, artist_id: artist.id) do |a|
                    a.name = spotify_album.name
                    a.release_date = spotify_album.release_date
                    a.number_of_tracks = spotify_album.tracks.count
                    a.image = spotify_album.images[0]["url"]
                    a.copyright = spotify_album.copyrights[0]["text"]
                    a.external_url = spotify_album.external_urls["spotify"]
                    a.label = spotify_album.label
        end
    end
```

It's always nice for a review app to have a star rating system!  In my app, star ratings either pertain to a user’s review of an album in their colleion, or are averaged across all reviews left by users that have the same album in their collection.  Also, the star ratings appear in several different locations throughout the app, so I decided to put the star review HTML and logic in a partial, `_stars` to which I send an integer value, whether that is from a user review, or an average of user reviews.  The  the filled and empty star graphics are courtesy of Font Awesome, which is a great resource to spiff up your app with cool icons.  

```
<% stars.times do%>
    <i class="fas fa-star" style="color:orange;"></i>
<% end%>
<% (5 - stars).times do %>
    <i class="far fa-star" style="color:orange;" ></i>
<% end%>  <%= stars %> out of 5
```

The partial is called with a local variable:

```
<%= render partial: "stars", locals: { stars: @review.stars } %>
```
or in the case of an average user rating:

```
<%= render partial: "reviews/stars", locals: { stars: @album.average_user_rating } %>
```

To find the average user rating, I wrote an Album instance method that joins the Review table to the UserAlbum table, finds all Reviews that users made of that album, and averages them.  Users are not obliged to review an album, so this is taken into account.  You can't round a nil value!

```
def average_user_rating
        if rating = Review.joins(:user_album).where("album_id = ?", self.id).average(:stars)
            rating.round
        else
            nil
        end   
end
```
I also had to write a lot of “button” logic into this app – always checking to see if an album was reviewed or had one to be edited, or yet to be added to the user's collection. So I created a ` _button` partial to handle that.

All in all, I had to flex a LOT of different coding muscles to create this app.  It was extremely helpful for me to review a lot of the Rails material a second time.  I also gained a greater understanding of working with Bootstrap, Font Awesome, and the Spotify API.  The Rails Project is a great opportunity to stretch your skills in all directions and build a solid foundation for what’s ahead!


![](https://i.imgur.com/T6qxRWA.png?1)

![](https://i.imgur.com/lTS8Aua.png?1)
