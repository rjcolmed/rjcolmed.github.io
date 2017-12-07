---
layout: post
title:      "Rails + OO Javascript + jQuery/AJAX"
date:       2017-12-07 16:21:46 +0000
permalink:  rails_oo_javascript_jquery_ajax
---


Flatiron really sold me on Rails. I love the platform and how easy it makes web development, especially the importance it places on convention (which is, it probably goes without saying, great for learning). 

But, neither Javascript nor jQuery, an external Javascript library, are baked into Rails. And yet, there I was, fresh out of Rails, on the precipice of a plunge into a section culminating in the union of a super conventional, organized framework and Javascript, a seemingly untamed, somewhat alien language. 

I was curious to see how Rails, Javascript, and jQuery would pair, and became even more so when the curriculum introduced AJAX. The more I progressed through the lessons leading up to this project, the more I started to see the full potential of throwing Rails, Javascript, and jQuery/AJAX into a code blender and seeing how the resulting mix could improve my project.

That project is Sādhanā, a somewhat contrived application that connects people who want to teach lessons related to dharmic-related practices (e.g. Yoga, Hinduism, Buddhism, etc.) to students who want to learn. 

jQuery and Javascript really improved Sādhanā from the user experience perspective. The main benefit in this respect was allowing users to retrieve information without disrupting their experience with a page refresh or redirect, both of which kind of annoyed me in the app's previous iteration. 

For example, I really didn't like that a teacher who wanted to see a list of their lessons would be redirected another page, a effect that's not really *that* burdensome, but still a bit annoying and a bit disconnected.

Here's another example in the same vein, code and all, of how jQuery + AJAX made my app more user-friendly:

On my teacher show page I have a button a user can click to see reviews students have left for them. To wire this up, I first created a Javascript class representing a review.

```
class Review {
  constructor(json){
    this.id = json.id;
    this.author = json.student;
    this.postTime = json.created_at;
    this.body = json.body;
  }
}
```

I used this as a base on which to build a series of prototype methods to handle the entire process of retrieving the data for, building, and displaying a list of the teacher's reviews.

```
Review.getSuccess = (json) => {
  let reviews = json.map(obj => {
    return new Review(obj);
  });

  Review.reviewTemplate = Handlebars.compile(Review.reviewTemplateSource);

  let reviewsHTML = '';
  reviews.forEach(review => {
    reviewsHTML += Review.reviewTemplate(review);
  });

  $('.ui.comments').html(reviewsHTML);
}

Review.reviewsClickListener = (event) => {
  event.preventDefault();

  let teacherId = $('.js-reviews').attr('data-id');

  $.get(`/teachers/${teacherId}/reviews.json`)
    .done(Review.getSuccess)
    .fail(Review.err);
}

Review.bindReviewsClickListener = () => {
  $('.js-reviews').click(Review.reviewsClickListener);
}

Review.ready = () => {
  Review.reviewTemplateSource = $('#review-template').html();
  Review.bindReviewsClickListener();
}

```

I also used Handlebars, which, as an earlier, template-less version of my code can attest, was a godsend for the purposed of organization and extendablity. 

Here's the corresponding Handlebars template:

```
<script id='review-template' type='text/x-handlebars-template'>
  <div class="comment">
      <a class="avatar">
        <img src="{{author.image}}">
      </a>
      <div class="content">
        <a class="author">{{authorName}}</a>
        <div class="metadata">
          <span class="date">{{formatPostTime}}</span>
        </div>
        <div class="text">
          {{body}}
        </div>
      </div>
    </div>
</script>
```


As you can hopefully see from the code above, I was able to leverage object-oriented Javascript, jQuery/AJAX, and Rails to better organize my code and, I think, make it more extendable. I can easily envison adding more methods to the Review class to deal with any review-related needs that might pop up in the future.



