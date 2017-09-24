---
title: The Office (US) Analysis
teaser: I was browsing through the subreddit, [/r/datasets](https://www.reddit.com/r/datasets/), for any datasets that were interesting and intrigued me. That's when I happily stumbled upon one containing every line from every episode of the show. Coincidentally, this show had been on my mind...
---

How it all began:
--------------------------------------
I was browsing through the subreddit, [/r/datasets](https://www.reddit.com/r/datasets/),
for any datasets that were interesting and intrigued me. That's when I happily stumbled
upon one containing every line from every episode of the show. Coincidentally, this show
had been on my mind ever since I recently found out that a three-part series reunion is
in the works! Rejoice! We can find out what happened to Michael and Holly (hearteyes),
Pam and Jim (#couplegoals), and Dwight (our favourite ex-assistant-(to-the)-manager).
However, the release date is set to be aiming for the summer of 2018. In order to
help with this long wait, I decided to perform an analysis on this data to see what
interesting things I could find. I thought this would be a nice and simple dataset to start with
for my very first post! Keep on reading to see what happened.

*If you're interested in this dataset as well, [here](https://www.reddit.com/r/datasets/comments/6yt3og/every_line_from_every_episode_of_the_office_us/)
is where I found it.*

Analysis:
--------------------------------------
The first thing I wanted to explore was the number of lines spoken by each character, and I thought
this would make for a very interesting visualization. I assume that Michael Scott would have the most number
of lines just because his character is one of the most important; however, Steve Carell (who played Michael)
left after season 7, so I wonder how this would affect the overall results. I assume Pam, Jim and Dwight
would also be in the top 10 at least.

In the same vein, I was also interested in the average length of lines for each character.
This is important because in general, a character who has long lines tends to be more important
as they contribute more to the plot and what they say might be deemed "more important". I also wondered
if there were any characters who may have had a lot of lines, but all of the lines were very short.

As with most data analysis done on Python, I imported the csv dataset into a pandas dataframe:

```python3
import pandas as pd
import re
filepath = '/Users/amandajylam/Documents/Projects/the-office-lines - scripts.csv'

df = pd.read_csv(filepath, header='infer')
df = df[df['deleted']==False]
```
The data was already formatted nicely, so I didn't have to do any data cleaning.
The data included lines from deleted scenes, but I decided that since these scenes were deleted,
I would exclude them from my analysis.

The next thing I did was set up all my variables to store my results:

```python3
result = {} #format: {season: {episode: episode_result}}
result_len = {} #format: {season: {episode: len_episode_result}}
for i in range(1,10):
    result[i] = {}
    result_len[i] = {}

episode_result = {} # format: {character: # of lines}
len_episode_result = {} #format: {character: length of lines}
```
Now, I created a helper function to help me with counting and keeping track of the count
for each character:
```python3
def count_character_lines(name, tracker, incrementer):
    if name in tracker.keys():
        tracker[name] += incrementer
    else:
        tracker[name] = incrementer
    return tracker
```

Then, I iterated through my dataframe:
```python3
old_episode = 1
for index, row in df.iterrows():
    if row['episode'] != old_episode:
        episode_result = {}
        len_episode_result = {}
    if ',' in row['speaker'] or '&' in row['speaker'] or ' and ' in row['speaker'] or '/' in row['speaker']:
        char = re.split(' and |[,&/]', row['speaker'])
        for name in char:
            episode_result = count_character_lines(name.strip(), episode_result, 1)
            len_episode_result = count_character_lines(name.strip(), len_episode_result, len(row['line_text'].split()))
    else:
        episode_result = count_character_lines(row['speaker'], episode_result, 1)
        len_episode_result = count_character_lines(row['speaker'], len_episode_result, len(row['line_text'].split()))

    result_len[row['season']][row['episode']] = len_episode_result
    result[row['season']][row['episode']] = episode_result
    old_episode = row['episode']
```

Now that I had all the information I needed organized, I needed to find the averages.
The first step was to sum all of the results from each episode:
```python3
total_sum = {}
for season in result.values():
    for episode in season.values():
        for character, count in episode.items():
            total_sum = count_character_lines(character, total_sum, count)
total_sum_len = {}
for season in result_len.values():
    for episode in season.values():
        for character, count in episode.items():
            total_sum_len = count_character_lines(character, total_sum_len, count)
```

The next thing I did was to count how many total episodes there were:
```python3
total_episodes = 0
for i in range(1, 10):
    total_episodes += len(result[i].keys())
```

Finally, now that we know there is a total of 186 episodes, we can get the averages:
```python3  
# averages in an episode
average_result_episode = dict((k, v/total_episodes) for k,v in total_sum.items())
average_result_episode = sorted(average_result_episode.items(), key=lambda x: x[1], reverse=True)[:25]
average_len_result_episode = dict((k, v/total_episodes) for k,v in total_sum_len.items())
average_len_result_episode = sorted(average_len_result_episode.items(), key=lambda x: x[1], reverse=True)[:25]
```
Note that since there were a lot of characters, both major and minor, I narrowed the results down to the
top twenty-five characters.

Here are the results graphed using the bokeh library:
![alt text][graphs/overall_num_lines_per_char.png]

Click [here] to zoom in and explore the graph further. (graphs/the_office_overall_result.html)

![alt text][graphs/num_lines_per_season.png]

Click [here] to zoom in and explore the graph further. (graphs/the_office_overall_season_results.html)

![alt text][graphs/avg_num_lines_per_episode.png]

Click [here] to zoom in and explore the graph further. (graphs/the_office_avg_num_lines_episode.html)

![alt text][graphs/avg_len_lines_per_episode.png]

Click [here] to zoom in and explore the graph further. (graphs/the_office_avg_len_lines_episode.html)

As is not surprising, Michael, Dwight, Jim and Pam have the most number of lines. I was a bit surprised about Andy, but he did become quite a major character after Michael left in season 7. Dwight has slightly more lines
than Jim - a competition that Dwight would surely be happy he one-upped Jim in. The other minor characters obviously had way less lines, especially those characters who were not recurring or brief guest stars. The more major characters
had at least double the total number of lines as the minor characters. I was a bit surprised that Kevin, Angela,
Oscar, and Phyllis said so little. In my mind, they are also an integral part of The Office, and maybe
I just thought they said a lot more than they really did. Stanley, Meredith and Creed, I expected, did not have
much lines to say - but the lines they did have were comedy gold and these minor recurring characters had a
consistent number of lines across the seasons.

Obviously from season 1 to 7, Michael had the most number of lines for each of those seasons. But it's interesting to see what happened when he left. Looking at Andy, there is a huge spike and he has the highest number of lines in season 8 compared to everyone. This is explained by the fact he became the new regional manager after Michael left. However, it is interesting to note that in season 9, his lines decrease by almost half. This is probably because in real life, Ed Helms, who had recently become very popular, was shooting multiple movies that year. Season 9 is dominated by Dwight, who has the most number of lines. Dwight, in fact, seems to have more lines than Jim in most seasons except for seasons 4 and 6.

Looking at the average length of lines in an episode, as usual, Michael, Dwight, Jim, Pam and Andy are in the
top five. However, it is interesting to see that Pam and Andy have almost the same average length of lines,
despite the fact Andy only became a prominent main character in season 8 and wasn't even in seasons 1 and 2. On the other hand,
Pam has consistently been a main character and actually had more overall lines than Andy. This is probably reflective of
Andy's verbose style of speaking and tendency to break out into song at inopportune times. 

Also, another interesting observation is that Nellie and Stanley have almost the same average length for lines, despite the fact that Nellie only appeared in seasons 8 and 9. However, if you know the character of Stanley - a quiet, gruff, no-nonsense man - this would make sense. He may have had more lines than Nellie, but when he speaks, he doesn't say much (he uses his face to convey his feelings [insert Stanley gif here]). On the other hand, Nellie, as some of you may remember, is quite an over-the-top person and loves talking. 

Another case is Phyllis, who has more lines than Kelly, Toby and Jan. However, they all have longer lines than Phyllis. This is explainable by Phyllis' quiet and timid nature, while Kelly is a non-stop chatter, and Jan has a forceful and strong personality. Toby is also pretty quiet, but Michael did needle and make fun of him a lot more than Phyllis. Another notable talker is Robert California who, even though had less lines than Gabe, Creed, Holly and Meredith, still had the longer average line in comparison.


That's What She Said!
---------------------------------------
Just for fun, I decided to count the number of "That's What She Said"s.

![alt text][graphs/num_twss_per_season.png]

Click [here] to zoom in and explore the graph further. (graphs/twss_season.html)

![alt text][graphs/num_twss_per_char.png]

Click [here] to zoom in and explore the graph further. (graphs/twss.html)

It is no surprise that Michael has said the most 'That's What She Said's in the entire series. I was a bit surprised by the low total counts, with the most number of TWSS in a season being 10. This means if there was one per episode, only about half of the episodes in a season contained 'That's What She Said'.

What I Could Have Done Better:
---------------------------------------
Looking back, I think that the measuring of the average length of lines could have been more accurate - using only
the number of episodes that those characters were in for specific averages for each character. However, I think that
my method also produced an interesting analysis: we can see that some minor characters who appeared on most episodes but don't say a whole lot may have a similar average line length as another character who may have appeared only a few times
but spoke much more.

Conclusion:
---------------------------------------
I hope you found this brief analysis as interesting as I did. Now, in order to fill the void of the long wait, don't mind me but I may go and binge the entire series again. I'll see you when I finally resurface from a Dunder Mifflin fog.
