---
layout: post
title:  "Falling Down in Zeros and Ones"
date:   2017-10-02 10:09:00 -0400
---

Making my FeaturedWiki CLI app was...operose.

And that word isn't, in this case, a stilted superlative. It perfectly encapsulates my experience making FeaturedWiki. 

But I did have a lot of fun. A lot. And I can't wait to do it again, to lie at the bottom of the metaphorical pit, waiting for the metaphorical pendulum to slowly slice at my metaphorical stomach while the metaphorical pain induces a weird metaphorical euphoria.

Less metaphorically, I coasted, dredged, and floated.

# Coasting

Starting off wasn't too bad. The project is, at its core, a customized version of the Student Scraper lab we were tasked to complete in the previous lab.

After a lot of hemming and hawing, I decided to make an app that presents users with blurbs from Wikipedia's "today's featured articles" (TFA). 

Here are 3 items I included in the app:

1. Today's featured article
2. This month's featured articles
3. A list of the most viewed featured articles

I got the basic skeleton of my app working fairly quickly. My basic UI was doing alright. My FeaturedWiki::CLI class was basic and fine. And I put the one metaprogramming technique I know to good use in my  FeaturedWiki::Article class' #initialize. Like so:

```
def initialize(hash)
      hash.each { |key, value| self.send("#{key}=", value) }
    end
```

Then...my final class creation, FeaturedWiki::Scraper

I spent a little time scoping out Wikipedia's homepage with Chrome's dev tools and scraped what I needed to construct the base of my featured article.

Here's the scraper method I ended up with:

```
  def self.scrape_featured_article_page
    doc = Nokogiri::HTML(open(BASE_URL + DATE_PATH))

    {
      title: doc.css("p a").first.text,
      blurb: doc.css("p").text.split(" (Full").first,
      url: HOME_URL + doc.css("p a").first["href"],
    }
  end
```

Note my use of the `BASE_URL` and `DATE_PATH` class constants. These helped me clean my code up a bit. Otherwise I would've had stringy URLs littered througout my code. 

Actually, `DATE_PATH` is kind of interesting, and it's the first solution in this project I had to research. Initially, I wrote a method to scrape Wikipedia's home page, specifically its 'From today's featured article' content block.

But, as I did a bit more research, I found that every featured article has its own TFA-specific page. And it's this page from which the content block on the home page is sourced. There's no direct link to it from the 'From today's featured article' content block on the home page, but (just FYI) you can get to it in a few clicks. 

This was great for me, because now, instead of scraping this content block on Wikipedia's crowded home page, I could scrape a stand-alone page containing only the information I needed. 

Back to the `DATE_PATH` constant. Each TFA page's URL follows this convention:

`https://en.wikipedia.org/wiki/Wikipedia:Today%27s_featured_article/MONTH-NAME_DAY,_YEAR`.

After some research, I found that Ruby has useful `Date` class that'd allow me to make each TFA URL's date path dynamic.

I called `Date`'s relevant methods...

```
"/#{Date.today.strftime("%B")}_#{Date.today.day},_#{Date.today.year}"
```

...and stored the result in my `DATE_PATH` constant for use in `.scrape_featured_article_page`. 

NB: the `strftime("%B")` call on `Date.today` allowed me to capture the date from `Date.today`'s usual output (`#<Date: 2017-10-01 ((2458028j,0s,0n),+0s,2299161j)>`) and convert the month's number—in this case 10—to the month's corresponding name, October. The `%B` flag is responsible for this.

I also used a simpler form of this in `.scrape_this_months_page`. Its URL path follows this convention: 

`https://en.wikipedia.org/wiki/Wikipedia:Today%27s_featured_article/MONTH-NAME_YEAR`. 

Thus my `MONTH_PATH` class constant and its corresponding value:  `"/#{Date.today.strftime("%B")}_#{Date.today.year}"`

Speaking of `.scrape_this_months_page`...
# Dredging

Here's where I ran into my first big struggle. Wikipedia's fault. 

In addition to giving each TFA article its own TFA-specific page, Wikipedia also has a page containing a queue of the entire month's featured articles. They plan them out in advance. Nice. That explains the TFA pages, now that I think about it.

I thought it would be nice for my app to present this queue to the user. So, a new FeaturedWiki::Scraper method. 

I don't want to spend to much time on this, but, suffice it to say that this page's HTML structure is...inconsistent. Getting what I needed was hard, but once I figured out the most consistent places to do so, everything turned out fine. 

A quick example of my pain: 

On this page, there's a little section for each day of the month's featured article. What there isn't is a dedicated article title element. What there was were two places from which I could possibly pull the title. Each article's blurb links to the full article via a link appearing fairly consistently in the blurb's first sentence. However, each link's `<a>` tags were inconsistently placed relative to its surrounding siblings (see screenshots below), so I had to come up with something else before I developed an anger-induced calcium deficiency. See ![screenshot A](https://imgur.com/wen3tVE) and ![screenshot B](https://imgur.com/fyk7tHp).

<blockquote class="imgur-embed-pub" lang="en" data-id="wen3tVE"><a href="//imgur.com/wen3tVE"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

After some putzing around with Chrome's dev console, Nokogiri, and Pry, I found another source for the title, one with more consistent placement relative to the document's structure. Each featured article section contains a **'(Full article...)'** link at the end its blurb. With `p.css('a:has(b)')[0]['href']`, I was able to access the article's title and URL to its full version. A twofer if I ever saw one (![screenshot](https://imgur.com/bzLtzsd)).

Wikipedia also has a page where they list the most viewed TFA articles. 

I wanted that for my app. Unfortunately, making it wasn't as easy as wanting it. 

Wikipedia's fault.

If you take a look at this ![screenshot](https://drive.google.com/open?id=0B78LezNCVoqHTU1FOWF5VFpjMWchttp://), you'll see that Wikipedia's editors organize each most viewed TFA and its respective data in a table. 

This was really hard for me to figure out how to scrape, especially since I only needed data from 3 out of 5 table columns. But, after a few rounds of stressful inspecting and Pry-ing, I sussed out the tags I needed and, even more importantly, was able to figure out how to iterate over the node structure to retrieve the info from those tags.

Here's a version of  `.scrape_most_viewed_page`:

```
def self.scrape_most_viewed_page
    doc = Nokogiri::HTML(open(BASE_URL + "/Most_viewed"))

    most_viewed = []
    tables = doc.css("table")
    tables.css("tr").each do |row|
      article = {}
      row.css("td").each_with_index do |col, i|
        case i
        when 0
          article[:title] = col.css("a").text
          article[:featured_date] = col.css("small").text
          article[:url] = "#{HOME_URL}#{col.css("a")[0]["href"]}"
        when 2
          article[:views] = "#{col.css("a").text} #{col.css("small").text}"
        when 3
          article[:blurb] = col.text
        end
      end
      most_viewed << article
    end
   most_viewed.drop(1)
end
```


As you can see, I'm iterating over each table (there were 3 on the page), then each row, then each column. 

I had to iterate over each row first—rather than over an array of columns generated with `css.(table tr td)` or an array of rows with `css.(table tr)`—so I could assign each column an index via `#each_with_index` (0 - 4) and scrape only those columns which contained the info I needed. If I hadn't done that, I wouldn't have been able to restart the columns' index count when I hit each row. 

Hopefully, the above gives you an idea of how hard making `.scrape_most_viewed_page` was for me. Fun, though!

# Floating


The rest of the app was comparatively easy relative to the work that went into `.scrape_most_viewed_page`. It involved a lot of fiddling around, but nothing that made me feel like I had a contusion. 

My one big takeaway from this project is that most of the heavy lifting was done at the interface level. Adapting my app to deal with Wikipedia's inconsistent HTML structure (which I don't blame them for) was tough, but cool. 

I'm happy I'm finished with this app, and learned a lot.

Cheers and on to the next.









