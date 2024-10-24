# Final-Project-Statistical-Modelling-with-Python

## Project/Goals
Use APIs to retrieve data, clean and explore that data, and assemble a reasonably functional statistical model. 

## Process
### CityBike Data
- picked a city (Auckland, NZ)
- explored the documentation for CityBike API to figure out how to get to the data; downloaded the Auckland bike station data
- extracted the json file
- started normalizing the json for usability and discovered that there are somehow no free bikes, no empty slots AND no users. It looked like the entire network was offline, so I picked a new city

- reran the query for Belfast, North Ireland
- normalized the data, tidied the table
- reran the entire function at 8:10 pm local time on a Saturday so that there would be actual usage on the network
- printed table to .csv

### Foursquare Data
- went through the exhaustive list of location categories to select a list 
- ran a test request and examined the output to get an idea what I would need to work with 
- created a function using a string formula to iterate through a list of coordinates corresponding to each bike station; tested this function returned the desired outputs with a dummy list of only the first two stations' coordinates to make sure the list was being iterated properly before doing the full call
- ran the full call, saved the return into a .json
- worked through extracting and cleaning the data; unnested the json, chose what columns to keep, selected venue tags for each location
- realized none of that matters when I went to check, out of curiousity, how many places had gotten picked up by multiple searches and realized that the same building id had wildly varying tags. One place was either a bar, restaurant, park or museum, somehow. (It's a pub, according to Google.) Somehow, inside my unpacking functions, the order of the rows had changed around. Had to redo **_everything_**.
- Redid the entire normalization and classification by hand, which took forever, but this time each place that got picked up multiple times only has ONE set of tags, which makes a lot more sense, so I guess that's a win. 
- printed the table to a .csv

### Yelp Data
- reviewed the API documentation and set up a key
- created new coordinate list compatible with the query format of the Yelp API, did a test call, then pulled the full call and saved it
- got a mentor to help me built a framework for an actually-functional unpacking function for the json because it was significantly larger than the last one
- realized I forgot to put the radius parameter and reran the call
- filled out the function framework with all the columns I wanted and got the data frame
- printed the table to a .csv

- for the 'top 10 restaurants section', got rid of location duplicates, then checked values for the 'rating' column; 5.0 was the highest rating, so I isolated those entries and classified entered classifications for them, using the same rules as I did for the Foursquare locations as much as possible
- merged final classification column back onto the rest of the 5 star information and pulled the restaurants specifically
- narrowed from 16 down to 5 using review count

### Data Joining
- added a static 'station number' column to the bike station table to use for the join, and a second one that would stay in place after the join
- joined the Foursquare data and the bike station data
- Got very stuck on data visualization; accidentally figure out how to make overlapping histograms, which worked out nicely for showing how much duplication there was in each dataset
- entered the three dataframes into SQLite 
- graphed available bikes vs empty slots, broken up across four for readability (and then merged into one in Paint because I couldn't get it to work with code and I was mad about it)
- graphed the mean, max and min of returned location distances from associated station 
- with difficulty and an assist from Alex, made a stacked bar graph showing the breakdown of location types associated with each station
- created an extra pre-joined CSV to pass over for the regression model, which I did not put into SQLite because it is redundant

### Model Building
- created a version of the joined table where the category column was numeric values
- generated dummy values for the categories and merged them onto the table to use for regression
- messed around with the regression function to figure out which value(s) to use to represent the stations; went through the process of removing variables with excessively high p-values
- decided on a final version of the model that was as good a fit as possible


## Results
CityBike's API is, as far as I can tell, not nearly as fine tuned as Foursquare's or Yelp's in terms of what you can request, but that might be due to the fact that once I realized I only had one network to content with and the results were grouped _by_ network, I stopped digging. But I didn't see a way to do a search by lat/long input. I actually had to go on google to find out how to do a seach that wasn't for the entire database; the documentation on the website itself is profoundly unhelpful. It seems to be talking about how to access the source code rather than the actual data. 

Foursquare's documentation is easier to understand, but Yelp's results are more detailed once you figure out how to get in. Which was not easy; it doesn't say anywhere that I saw that you have to have 'Bearer' as part of your header, and if you want to pull the category list, you need to access the developer beta. In both cases the granularity of the data makes genralization difficult; most locations have multiple classifications to tease apart in order to get the broad sense of a place. Is it a bar-bar, or a restaurant that **has** a bar? Both can end up tagged with 'bar', but they are two very different establishments, and looking it up for every single one defeats the purpose of labelling them in the first place. I also found a baffling one where a football pitch was labeled as a movie theater, which I have no explanation for.  

There isn't a way to generate a model from the data I had that was really _good_. There were ones that were iffy and ones that were _bad_. Using station numbers rather than lat/long in the empty slots model (being sure to keep the constant in) I ended up with only station number, distance from station and libraries, with the largest coefficient by far being the positive library coefficient, which has the very funny implication that the presence of libraries compels people to take bikes to ride to them, even at 8 pm. 

## Challenges 
I had not figured out how to use the normalize function directly to unpack a json into a table while accessing multiple columns containing nested data and without losing higher levels' data while working through the Foursquare data, but know I probably know how to do it correcly with a function. Before, trying to use normalize, sometimes the data from higher up disappeared, sometimes the table was the right shape but filled with NaNs, usually the cell failed to run. Doing it in a function with a loop didn't work very well either, and then it turned out the awkward hand holding function I was trying to use was shuffling the data around, so I had to do all of it manually, level by level anyways, and a full day of work was wasted.  

I also could not figure out a way to go through FourSquare's incredibly redundant and overly complicated location classification system using code that wouldn't be filled with contradictions, so I did it the long way. And even then I had to Google some of the places; one was 'Organization', which was not helpful. I just did not put classifications on the Yelp returns until I got to the 'top restaurants' section, and then I only did places that were rated the highest possible. 

I had a miserable time trying to get data into graphs. Looking up how-to's didn't help, because they didn't explain which parts of the code were doing the bits I was interested in, so I had no way to try and replicate it, and then the way I was wanting to transform the data was making the columns I needed into inaccessible indexes, I think? Some kind of awful multindex was happening and I gave up and moved to the SQLite section. It was well past mentor hours because the timezones are not in my favor. 

## Future Goals
If I had endless hours to spend on debugging and checking the outcomes, it would be interesting to see how long it would actually take to create a function that could get the results I want for the category tagging. It would be a lot easier if the parent tag was attached, especially with Yelp. Although with Yelp, now that I think about it, it might be possible to use the parent categories from the category info to try and do categorization that way. 

There's also data in the Yelp database that I think could have been useful in the statistical model, like rating and review count. Not to mention doing a larger pull that goes beyond the default return count. Foursquare gave me 10 per station, but that's probably not everything. I could also possibly expand the search and just grab even more kinds of locations. 

I also wonder if pulls at earlier times of the day would give results that could be modelled more effectively; 8 pm isn't exactly peak time for most activities, which could be messing with things. As could the time of year; I don't know how dedicated people in Belfast are to using the cycling system. 
