# This all needs to be updated for 2018


This is a Tarbell project that takes a [complicated Google spreadsheet](https://docs.google.com/spreadsheets/d/1FIl_mXfBWeFonUBK8OVK3HvQgS4SmsP406h90xpcS24/edit#gid=1743693722) and generates form charts. [Live in the wild.](http://registerguard.com/rg/sports/trackandfield/35651655-345/2017-outdoor-ncaa-track--field-championship-form-charts.html.csp) Here's what I did from start to finish.

# Blueprint update (July 2017)

I created a blueprint for form charts and have integrated that technology so this can act as an example. I also added a `raw` directory for easier copy/pasting into DT. This required me to create an includes folder where all of the content went. I will try to update the documentation below to the best of my ability but there may be inconsistencies.

See registerguard/form-chart-blueprint for more on creating a new project from a blueprint. 

# Before you begin

Get the form charts copy edited.

Do it.

In 2017 I thought they had been edited but they weren't. Then I had to go back after publishing and re-generate and DT does not like that. Better yet, this should probably be a flat page with minimal header baked to S3. But still make sure they get copy edited.

# Getting data in order

First, Mark created two DT files (one for men's and one for women's). copied everything out of those files and did some cleaning. I got a list of all of the events together and with some regex I got tabs separating the name, time and day. Then I put them into the [events sheet](https://docs.google.com/spreadsheets/d/1FIl_mXfBWeFonUBK8OVK3HvQgS4SmsP406h90xpcS24/edit#gid=0). I added tags for gender, event type and day of week. I also added semi-logical slugs for each event. (I also created a display column whihc says whether or not the form chart is open or not. More on that later.)

With this list, I made a new sheet for each event, using the slug as the name. It's important to note that you must use underscores as Tarbell does not like hyphens or spaces. This is critical because later we will be looping over the events sheet to get the names of other sheets.

Now, the time-consuming part: Copy and paste each event into its sheet. This can be sped up if you regex and get each event tab separated before copy/pasting. Some events had extra people, I limited each event to eight places, this will be important later.

The order of the sheets doesn't matter, but the order of events in the events sheet does. I opened with the team projected scores and then did chronological. I thought about doing reverse chronological since the Oregon-heavy events were all the last two days, but thought that might lead to some UX jank. People will probably scroll down for the "next" event instinctually but they would have to scroll up instead. With the filter buttons and everything collapsed by default this shortens up the page and acts like a schedule so we'll call it good.

Ok, now that we have all the data in place, let's work on the template.

# Tarbell template.

This project is very simple (I ~should~ did make a Blueprint). The `_blueprint/_base.html` contains all the shell, `/index.html` and `raw/index.html` act as middlemen and the goods are in the `_includes` directory.

## _base.html

### Extra CSS

This gets built into the template after Bulldog. BUT should be included in WebHTML before content if you put this junk in DT.

### Content

This is where the goods go.

### Extra JS

This gets built into the template after jQuery loads. BUT should be included in WebHTML after content if you put this junk in DT.

## index.html

The index file has those three parts included.

## _includes

This directory holds all the content. The filenames are pretty self explanatory. 

### _content.html

#### macro event()

This macro holds the inner loop of this project that loops over each event table. The key to this is: `{% for s in g.current_site.data[eslug] %}`. I had to reach out to Chris Amico because I was having trouble with a loop within a loop. He turned me on to the `g.current_site.data[variable_that_holds_sheet_name]`. That `eslug` variable gets passed in and contains the specific event slug.

Inside that loop there's a big conditional to spit out different HTML depending on which row you hit. We open with analysis, even though that is the last row because it's the most unique. Then we move to the first row (which is really the first row after the header) to open up the table. Then row eight (remember when I said I had to chop some of these down to eight?) to print out the close of the table. Then we have the else which gets rows two through seven.

And the loop ends.

#### Content

The content is wrapped with #story. This makes it easier to copy/paste into DT. We open with some text, then move into the filter buttons. The IDs on those are used by jQuery later.

Then we go into the form charts. We loop over each e in events and create a button that can toggle() and test to see if it's a team scores or event. This is because the team events don't have days so we don't want empty parens.

Then we have open up the event div container, add the slug as id, the appropriate tags and whether or not it should be open by default.

Next we hit the macro and pass through the slug. Then we close everything out.

### _css.html

This contains CSS specific to this project. I style the red buttons and use flexbox to align them. Then I style the event buttons and the event tables. I don't think there's anything terribly complicated in there.



### _js.html

Here is where we include any project-specific JS. I include jQuery if it hasn't been loaded already. In the template it will be loaded, but if we copy into DT, it wouldn't be. 

Then we get into the show/hide logic. Basically, you can cut the data three (or four - we don't use event type) ways. Show all or hide all. Show men or women. Show a specific day of the week. If someone chooses one of those, it will disappear others. BUT if someone selects a filter then scrolls, they can open or close more and the filter does not revert.

And we're done.

# Shovel it

Ok, unless things have changed, the procedure for stuff like this is to shovel it over into DT. 

I created a story file called web.sp.ncaaform.0607. Gave it all the proper metadata and turned off comments.

I matched the WebHeadline and byline to what I had in Tarbell. The next part is a bit wonky.

With the Tarbell project up in my browser, I reload the page then search the source for a comment like `ncaa-copy` and there should be six. These are only there to allow you to grab the three bits and combine them into one file.

![screen shot 2017-06-07 at 2 08 27 pm](https://user-images.githubusercontent.com/4853944/26901491-e70a0680-4b8a-11e7-97dc-988a28bb524b.png)

So grab the CSS, content and JS and paste them all into a text file. I minified this manually to make it easier to manage. I'm also fairly confident that DT does not like 4,400 lines of code being dropped into InCopy. So un-tab all the lines and replace `\n` for nothing. That's a 30 percent reduction in file size, which helps a little but DT still kicked like a mule. I tried saving a few times and the story would break.

Once you get the stuff in there, slot it to sports picks with a Special full story template. That should give you an idea of 345 in sports. Open that up and if it looks good, grab the URL and paste that into the WebLink field.

I think that's it. Good luck.

# Getting started with Tarbell

See: tarbell.readthedocs.io

Generally speaking I would advise the following:

```
mkvirtualenv tarbell
pip install tarbell
cd ~/projects/...
git clone ...
tarbell serve
```

Then duplicate the Google sheet, crack open the tarbell_config and replace the old id with your new one. See if that works. Then edit the data or template as needed.

