---
layout: post
title:      "Rails? Rails and STI"
date:       2017-11-16 16:26:41 +0000
permalink:  rails_rails_and_sti
---


I had a lot of trouble deciding what exactly I was going to do for my Rails project. In the lead up, Rails seemed to me an exceptionally powerful weapon, and I knew that, whatever domain I pointed it at, it'd help me produce something good.

But I couldn't settle on a domain. It took me forever. And when I finally did pick one, a simple clone of [NetrunnerDB](https://netrunnerdb.com/), I abandoned it a few days later because I wanted to do it justice, and doing so would take me months. Not great, especially considering that we bootcamp students are all on a but of a time crunch (and time is money, after all). 

I then briefly considered Rails-ifying my Sinatra app, which needs work and more functionality, but, after a conversation with one of our instructors, decided that it'd be better if I flex what I learned so far on a new domain.

To that end, I came up with a new-ish doman: Sādhanā, a simple web app for practitioners and teachers of Hinduism-based philosophies. It's kind of like a Craiglist for yogis. Teachers post classes, tag them whatever they want, and students can enroll in classes.

Anyway, as with my last post on the subject of projects, rather than reviewing the entire thing, I'm going to focus on one aspect of my learning experience, Single Table Inheritance, or STI.

# I'm riddled with STIs.
Just kidding. On all fronts. I used STI once, more as a matter of curiousity than real need. Before I get into how and why I used STI in Sādhanā, let me briefly go over what it is.

STI allows you to create multiple tables (and by proxy, models) that inherit from a single table (model). With that inheritance comes access to all the columns (attributes), validations, etc. of the parent model. STI is especially useful if your project calls for several conceptually similar, but slightly different types of data representations.

In my app, I have teachers and students. They're both technically users and therefore share a lot of (in my case all) the same attributes, so I could have created a users table and model, added a column to users table that indicated the type of user (maybe a 'teacher' column with a boolean data type), and maybe a few helper methods to check if the logged-in user has a true or false in their teacher column. 

I thought about this a bit. There's nothing wrong with doing that (and it's probably a lot faster, execution-wise), but I'd just read a few articles on STI and wanted to see if I could incorporate it into my project.

To use STI, you have to add a `type` column to your soon-to-be-a-parent table and make sure the models you want to be children inherit from the parent model. Here's what I did:

```
###My migration looked something like this:

class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :first_name
      t.string :last_name
      t.string :username
      t.string :email
      t.string :password_digest
			t.string :type #=>this is the important bit

      t.timestamps
    end
  end
end

###My models look something like this (I removed most of the validations, methods, etc. they accumulated over the course of the project to make this clearer:

class User < ApplicationRecord #=>This is the parent model
  has_secure_password

  validates :first_name, :last_name, :email, :username, presence: true

  def full_name
    "#{first_name} #{last_name}"
  end
end

class Student < User #=>Inherits from User
 
end

class Teacher < User #=> Also inherits from Teacher

end

```

# What I learned from using STI
Using STI in Sādhanā allowed me to abstract away the conceptually different, yet data-layer-ceptually same Teacher and Student models. On a very shallow level, it allowed me to semanticize (I'm making up too many words here) the code-level usage of users, to wit:

Writing `@teachers = Teacher.all` or `@students = Student.all` in their separate, respective controllers, rather than having one Users controller serving both of them, potentially cluttering up its actions with calls to User's scope methods like `User.where(teacher: false)` to get all students or something like that.


In fact, when using STI, these two methods are technically the same: 

`@students = User.where(type: 'Student') versus @students = Student.all`

Also, since the Teacher and Student model inherit from the User model, any macro (like has_secure_password), validation (like validates :name, presence: true), or method (like #full_name in my example above) are available to them.

```
## I can do this:

Student.create(first_name: "Jomney", last_name: "Bimbert")
Teacher.create(first_name: "Dana", last_name: "Scully")

student = Student.last

student.first_name => "Jomney Bimbert"

# and this:

teacher = Teacher.last

teacher.first_name => "Dana Scully"

Student.all => Student ActiveRecord object representing Jomney
Teacher.all => Teacher ActiveRecord object representing Dana

## And, if I were so inclined:

User.all => [ ActiveRecord object representing Jomney, ActiveRecord object representing Dana]

```

# Getting myself cleaned up

I think it's cool that STI allowed me to clean up my code a bit, and better express the conceptual separation between students and teachers, making (other bad practices littering my code notwithstanding) my code more readable. 

Having read a few articles on the subject, you should exercise caution when using STI, because it could cause problems as your application grows, among other things. But, it's a tool and it served me well in this project. Did I need it? Probably not. Was it interesting to use? Absolutely.

If you'd like to learn more about STI, [here's a great post ](http://eewang.github.io/blog/2013/03/12/how-and-when-to-use-single-table-inheritance-in-rails/) on the subject (written by a Flatiron alumnus) and [here's a link](http://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html) to a skeletal Ruby on Rails doc page.

Enjoy.




