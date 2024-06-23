# Data Sourcing Challenge

## An exercise in:
- Pulling data from external (i.e. world wide web) databases
- Using APIs
- Extracting and cleaning data
- Merging datasets

## Notes
Looking at the output from the original starter code, I noticed that some movie titles were not properly extracted from the review headlines.  
Example:  
- **Headline:** `Review: ‘What’s Love Got to Do With It?’ Probably a Lot`  
- **Expected title:** `What’s Love Got to Do With It?`  
- **Extracted title:** `What’s Love Got to Do With It?’ Probably a Lo`  

I'm not even sure what they did to cut it off at 'Probably a Lo'. I'm not a pro at Python regular expressions but I still think I can do better than that. My approach was to use three regex patterns in three stages.

### Stage 1
The titles are always between \u2018 and \u2019 (left and right apostrophe), so we have to capture text between those two characters.  
If we don't use a 'greedy' quantifier, it will capture the text between the first \u2018 and the *next* \u2019.  
- **Headline:** `Review: ‘What’s Love Got to Do With It?’ Probably a Lot`  
- **Expected title:** `What’s Love Got to Do With It?`  
- **Extracted title:** `What`  

Therefore, we use `re.compile(r" ?\u2018(.*)\u2019 ?")`, which is capturing all text between the first \u2018 and the *last* \u2019, whether or not there is a space before \u2018 or after \u2019.

This presents a new, but fixable problem:  
- **Headline:** `‘About Fate’ Review: Love the One You’re With?`  
- **Expected title:** `About Fate`  
- **Extracted title:** `About Fate’ Review: Love the One You`  

The last \u2019 was not terminating the movie title, but was later in the headline, so it captured more than we wanted.  

### Stage 2
Python doesn't know which \u2019 marks the end of the title. But we know that it marks the end of the title if no text comes after it, or if the next thing after it is a space followed by more text. So the next thing to do is to capture only the text before \u2019 if \u2019 is followed by a space and more text.

Therefore, we use `re.compile(r"^(.*?)(?:’ .+)?$")`, which is using a non-greedy quantifier to capture:  
- the starting text `^(.*?)`  
- up until any \u2019 that happens to be followed by a space and more text `(?:’ .+)`  
- if that's how the rest of the text goes `?$`

### Stage 3
It's almost perfect, but while all authors are putting the movie titles between quotes, some chose to include a comma before the end quote. Similar to stage 2, we use `re.compile(r"^(.*?),?$")` to capture:  
- the starting text `^(.*?)`  
- up until any comma `,`  
- if that's how the rest of the text goes `?$`

### Rate limits
The homework asks that we include an unspecified cooldown after every 50th request when querying TMDB. TMDB's rate limit is 40 requests every 10 seconds. That said, a cooldown every 50 requests seems arbitrarily chosen. Perhaps a better approach (but keeping it simple) would be to sleep 0.25 seconds every loop. With a 1/4-second delay, plus the processing time in the loop, it would be impossible to exceed the rate limit while keeping things moving at a reasonable speed.  
In my case, with the for loop processes included, my machine can't make requests faster than the rate limit anyway, so no sleeps are needed in the first place.
