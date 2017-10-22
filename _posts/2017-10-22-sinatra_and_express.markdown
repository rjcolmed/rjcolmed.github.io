---
layout: post
title:      "Sinatra and Express?"
date:       2017-10-22 20:32:33 +0000
permalink:  sinatra_and_express
---



I've just finished my first Sinatra app (meaning, I took a break from endlessly tweaking its CSS).

It was (and still is, until the CSS-mania unhooks its unholy talons from my brain) a big, involved project; so much so that writing a blog post on it seems just as daunting as starting on the project itself.

Having said that, I'm going to try and limit the scope of this post to one aspect of my "journey" so I don't spend the next year writing it: The similarities and differences I noticed (with my untrained eye) between the Ruby/Sinatra and Javascript/Express frameworks.

To start, I should mention that I took a Web Development course on Udemy.com before I joined Flatiron School. It was on sale for $10 and I finished it in about 3 weeks. While I'm learning **much** more here, major, major props to Udemy and the instructor, Colt Steele, for offering a great course for an amazing price. It was a great primer for Flatiron and I highly recommend checking it out as a supplement to Flatiron's curriculum. 

Anyhow, the culminating project of this course was YelpCamp, a fully-functioning, RESTful CRUD app built with Javascript, Express, Node.js, and MongoDB. 

A quick note: I'm not going to really touch on the differences between MongoDB and SQLite. Suffice it to say, MongoDB is a non-relational database, as opposed to SQLite, which as all us students know, is relational. 

YelpCamp was pretty much just a code along, so I can't say much of it stuck with me after the course. But, right before I started here at Flatiron, I tried my hand at making my own web app based on YelpCamp. That project was a yoga sequence maker I called Āsanāḥ, which is Sanskrit for "poses" (in the context of physically oriented yoga practice).

I got about half-way through with Āsanāḥ before I decided to set it aside and dedicate my full attention to Flatiron's curriculum. And as soon as we started up on CRUD and RESTful routing, I began to experience a bit of déjà vu.

# Project Structure

There are quite a few similarities shared between Express and Sinatra the in this respect. 

My Express project had a Package.json file, which to my eyes is analagous to Gemfile.lock in that both are records of the last working versions of all dependencies used in your project. I think the Package.json file also serves a similar purpose as Gemfiles in general. And both are the product of NPM (Node Package Manager) and Bundler, respectively. These two programs (I'll call them) are spiritually the same, too. They each retrieve and maintain libraries (modules or gems, if you like) for projects in which they're involved.

My Express app had a `node_modules` directory, which, to me, looks like a folder full of what would be gems in Ruby-land. I suspect that this folder could live somewhere else on your drive during development, rather than in your app's directory, and your end-users would invoke NPM to download all of your project's dependencies listed in Package.json (much like Bundler, Gemfile, and Gemfile.lock work together in a Ruby-based project).

Both projects I worked on had views, public, and models directories, which is probably a result of the MVC paradigm we all, as Flatiron students, are familiar with. *(I probably would have had a controllers file in the Express/JS version of Āsanāḥ, had I known about separation of concerns, etc. All of my routes in for that version were squeezed together in a massive controller. In fact, separation of concerns accounts for most of the broad differences in file structure between the two projects, I think.)*

# Listening on Express vs. Sinatra

I'm not sure where or what the Express Āsanāḥ's corresponding config.ru file would be. I'm not even sure if there is one. Hazarding another guess, this is probably down to the differences between Rack/Sinatra and Express.

The difference I could see (again, with untrained eyes) is in how one connects to a server. In my Express app, I included the following route in the application controller, explicitally designating the connection port:

```
var app = express();

app.listen(3001, function(){
    #code goes here. I inserted a console.log to print a visual reference to the connection in my terminal
    console.log("#############################################");
    console.log("Now running on port 3001")
});
```

Note: The first line is an instatiation of Express in a variable named (conventionally, I think) "app." 

Rack seems to do all this behind the scenes, implicitly.

This kind of brings me to the big similarity between the two projects (and, by exntension, simple web apps built on the two frameworks).


# CRUDdy REST

Routes in Sinatra and Express are by and large the same, at least (probably) in simple web apps like Āsanāḥ. For example, note the similarities between calls to the root route:

```

#Express

app.get("/", function(req, res){
    res.render("index");
});

#Sinatra

get '/' do

    erb :index
end
```

They look pretty similar! The differences are, I'm guessing, down to each framework's base language's syntax. Express calls its `get` method on `app` (which, remember, is kind of an instantiation of Express) with `/` passed in, and triggers a callback function accepting two parameters. The first parameter is `req`, whatever info was transmitted with the request, and the second, `res`, the route's response. In this case, the route responds by rendering the index page. 

`req` comes into play when you need to retrieve info, like paramenter sent along with a `post` request. 

Here's a `post` route in Express and Sinatra:

```

#Express
app.post("/sequence", function(req, res) {
    Sequence.create(req.body.sequence, function(err, newSequence) {
        if(err){
            res.redirect("/");
        } else {
            newSequence.save();
            res.redirect("/sequence");
        }
    });
});

#Sinatra
post '/sequences' do
    sequence = Sequence.new(params[:sequence])
    sequence.asana_ids = params[:asanas]
    sequence.user = current_user

    if sequence.save
      redirect "/sequences/#{sequence.slug}"
    else
      redirect 'sequences/new'
    end

  end

```

Fairly similar. Again, a major difference is how Express (based on Javascript's structure, likely) transmits (and allows you to act on) info via callback functions. 

Also, note how each deals with information received from the `post` request. Express sends info, extracted from `req`, to the call back function. That information can then be acted on, like in the call to my Sequence model above. 

Express's `req` seems functionally similar, maybe even analogous, to Sinatra's `params`.

# What I Learned

Without sinking into a deeper comparison between the two frameworks, it seems pretty clear that most of their structural similarities result from conventions, namely CRUD and RESTful routing. 

Besides making me more comfortable with these conventions (and with Ruby, Rack, Sinatra), working on this project in two frameworks really helped me understand how crucially indispensable conventions are—I (a virtual layman) was able to recognize similarities between code written on two different frameworks, and enrich my general knowledge of coding, because each framework follows broader conventions. 

This gives me a lot of hope and encouragement for my learning experience moving forward. I'd really recommend trying to port this project over to another, non-Ruby framework!



