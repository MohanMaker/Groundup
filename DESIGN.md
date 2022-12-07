## Database:
The central component of the backend is a database called 'groundup.db'. This SQL database has three tables. The first table, called users, remembers the username and password of a user from when they registered. It also records in the type column if the user is a data collector or client, based on which registration form was used. This allows the user to be redirected to the client or the collector dashboard when they login. The second table, called data collectors, stores information about all the data collectors that are currently registered. This table holds the key facts about the data collector registered, including id, name, latitude and longitude, occupation, degree, and sector. Additionally, the table has a foreign key "userid" which references the id in the users table. This allows us to associate each data collector with a login, so the correct data collector information can be presented when logging into the collector dashboard. Finally, there is a very similar table called datacollectorsfiltered. This holds the same information as datacollectors (minus the userid foreign key), but is used to generate the map page. When the user filters data collectors based on certain parameters, the resulting data collectors are duplicated from the datacollectors database into the datacollectorsfiltered database. The map endpoint, when called, then draws the data collectors from the datacollectorsfiltered database, allowing us to only show certain filtered data collectors.

## Map:
One of the core features of our project was an interactive map displaying data collectors to prospective clients. There were three considerations while creating the map. First, we wanted it to center around India. This was done using standard functions from the Folium library. Second, we wanted to create markers on the map for each person in the data collector database. We used a helper function called 'popup_html.py' to create a popup that shows the data collectors name, education, sector, and occupation, drawing values from our SQL database using db.execute, and plotting with Folium. We originally created the map’s functionality in JavaScript, and were trying to use PHP and the Leaflet library to query the database and add markers. However, when this wasn't working, we switched to Folium which allowed us to create the map using Python ('app.py'), thus making it easier to connect the map and the database. The third goal of the map was to filter out information. We filter the datacollectors database table based on the user inputs in the client dashboard, saving the filtered values to a table called datacollectorsfiltered, which is was the map endpoint graphs. This also required us to make failsafes for the many different types of filters possible.

One issue we faced was with map layout and design. It was difficult to display the map on an html page that included a bootstrap navbar. In Folium, every time the map is rendered, it overwrites 'mapdata.html' with the new map. To solve this, extend 'mapdata.html', in the bottom of another file called 'map.html' that head the header code. 

## App.py
### Index
The heart of 'app.py' is the index function. The function has two parts, one for when the logged in user is a client, and one for when the user is a data collector. The type of user currently logged in is stored with flask session.

If the current user is a client (data seeker), then app.py renders the client homepage (client.html) and populates the dropdown menus in client.html with all of the unique occupations, educations, and sectors of the data collectors currently in the database. This allows the client to filter for data collectors based on location, occupation, education, and sector using dropdowns and text/number input boxes. When the client submits the filter form, the data is sent to 'app.py' through post. The function then needs searches for the data collectors that fit into the filter parameters inputted. This is tricky, because if the client only filtered certain categories, only these categories should be included in the search. Or, if the client just clicks “filter” while leaving all the filters blank, then all of the data collectors should be rendered (no filters applied). To accomplish this complex request, we dynamically generate a SQL search query based on the filters which the user has selected. 

If the current user is a data collector, then app.py renders the collector dashboard (collector.html). This shows the current data collector, including their username, location, occupation, degree, and sector. Collector.html also has a series of form inputs which can be used to edit the specific characteristics of the data collector in the database. When this form is submitted via POST, 'app,py' updates the collector in the database, and re-renders template with the new values.

### Register, Login, Forgot Password 
App.py handles logging in, registering, and changing the password with three different methods. The register function detects whether the user registers with the data collector registration form or the client registration form and updates the database with the username, password hash, and user type accordingly. Login then detects which type of user the login credentials correspond to, and if the user is a new data collector, it creates a blank datacollector that has a foreign key linked to the login username. The user can then update the name, location, etc. of the data collector in the data collector profile. 

Forgot password allows a user to reset their password if they know their username. The html elements are all in login.html, however, the forgot password button opens up in a popup form. The inputs of this form (username and password) ‘POST’ to /forgot, a separate function in 'app.py'.. If the inputted username matches an existing username, a SQL query overwrites the existing password with the newly inputted password using UPDATE. Once that’s done, the user can log in as they normally would with the new password. 

## Helpers.py
Helpers.py contains a few functions which assist the functions in app.py. Namely it uses the Nominatim geocoder from GeoPy to translate latitude and longitude coordinates into addresses and vice-versa. This is used for inputting and displaying locations on the dashboards and the map. We initially tried to use an API to do this, but many APIs limit the number of queries, so we ended up using a Python library with the same functionality. 

## HTML, CSS, Mobile-Friendly
Another design decision involved the register, login, and dashboard pages. These pages use tables to format the information on the website into two columns. However, on mobile, the right side columns were being placed outside of the page. We updated the CSS to automatically reformat the table to fit into one column when the page reaches a certain size. 