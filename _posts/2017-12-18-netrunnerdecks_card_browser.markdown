---
layout: post
title:      "NetrunnerDecks Card Browser"
date:       2017-12-18 20:55:13 +0000
permalink:  netrunnerdecks_card_browser
---


For my final project, I decided to tackle something connected to a domain I'd been wanting to involve myself in for a little while now, Netrunner and LCG (living card games). 

Netrunner's an incredibly fun LCG I used to play a lot a few years ago. It's kind of like Magic the Gathering, but set in a "cyberpunk" dystopia where, predictably, huge corporations battle against hacker-types for whatever reason. One player plays as the corporation, and one as the runner (a hacker trying to break into the corporation's server). That's sort of a simple explanation of what Netrunner is, but it's enough background to get started on this post about my project, I think.

NetrunnerDecks Card Browser (a long name, I know), is a very simple SPA (single page app) allowing users to sign up for an account, browse through the current library of Netrunner cards, and add cards they like to their own favorites list.

Rather than talk through the entire process of making this app, I thought I'd highlight what adding authentication to this app entailed for me. I think this is an interesting topic, especialy for novice coders, because it involves the integration of four frameworks/libraries: Rails, React, Redux, and JWT. 

NB: I relied rather heavily on Sophie DeBenedetto's [blog post](http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/) on this subject, and HIGHLY encourage anyone interested in adding authentication to their Rails/React/Redux app to read it thoroughly. My post on the subject isn't meant to be a walkthrough. Rather, it's a simple "listing-off" of how I used the aforementioned mix of frameworks/libraries  to expand the functionality of my SPA.

#JWT and Rails

To start, I added the `jwt` gem to my Gemfile, ran `bundle install` and created an `auth.rb` file in the `lib` directory of my Rails API.

Here's the content of my `auth.rb` file with a few inline notes explaining a few of the key components therein.

```
require 'jwt'

class Auth

    ALGORITHM = 'HS256' //=> this class constant informs jwt what kind of algorithm to use when encoding and decoding the information you're passing between your API and client.

    def self.issue(payload)  //=> Here in .issue and in .decode below, I'm passing the algorithm, payload (what I want to encrypt, and a secret (which I created using Ruby's built-in Digest object and stored in my environment as AUTH_SECRET
      JWT.encode(
        payload,
        auth_secret,
        ALGORITHM
      )

    end

    def self.decode(token)
      JWT.decode(
        token,
        auth_secret,
        true,
        { algorithm: ALGORITHM }
      ).first
    end

    def self.auth_secret
      ENV["AUTH_SECRET"]
    end
end
```

Everything `.encode` and `.decode` is need by `jwt` to...encode and decode information passing between your API and your client.

`jwt` uses these three components to construct a rather large token (a plain text string, essentially) that you can add to your HTTP request headers whenever you want to pass protected information to and fro.

This is very similar to OAuth tokens and the like.

For an example of how to use this for, say, user log in, see the code in my API's SessionController:

```
class SessionsController < ApplicationController


  def create
    user = User.find_by(username: sessions_params[:username])

    if user && user.authenticate(sessions_params[:password])
      jwt = Auth.issue({ user: user.id })

      render json: { jwt: jwt }
    else
      render json: { message: user.errors }, status: 400
    end
  end

  private

  def sessions_params
    params.require(:user).permit(:username, :password)
  end
end
```

As you can see in #create, I take in the params sent to me from a client-side form (made via a POST request to my SessionsController and authenticate them via Bcrypt (nothing new there). `jwt` comes into play when my user is authenticated. 

My API knows the user is who they say they are, but my client (in this case, my React/Redux app) has no idea what's going on. The beauty of `jwt` is that I can encrypt the user's id into a token and send it back as a JSON response to my React/Redux client. Noice.

#Now We're in React/Redux

Oooook. Below is the code for my app's SessionsAPI and its corresponding sessions-related action This is cool design pattern I picked up from [Sophie's blog post](http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/). You basically move all calls to external API's into their own class declarations, and make calls from that object wherever you need to in your app. All you have to do is import the code into whichever file you need to make the calls from. 

```
## in session_api.js

import fetch from 'isomorphic-fetch';

class SessionAPI {
  static login(credentials) {
    const request = new Request('/login', {
      method: 'post',
      headers: new Headers({
        'Content-Type': 'application/json'
      }),
      body: JSON.stringify( { user: credentials })
    });

    return fetch(request)
      .then(response => response.json())
      .catch(err => err);
  }
}

export default SessionAPI;
```

```
## in session_actions.js

export const loginSuccess = () => {
  return { 
    type: types.LOG_IN_SUCCESS 
  }
}

export const logInUser = (credentials) => {
  return dispatch => {
    return SessionApi.login(credentials) <============here's where SessionApi.login() is called
    .then(response => {
      sessionStorage.setItem('jwt', response.jwt); <==== here's where I store the jwt token for later use
      dispatch(loginSuccess()); <=== here's where I dispatch an action to my session's reducer (see below)
    })
    .catch(error => console.log(error));
  }
}
```

I attached SessionAPI.login to a click listener on my client's LogInForm componen. When that fires, it kicks off a POST request that sends the user's credentials (which they inputted via the login form) to my Rails API's SessionsController. 

There, as I touched on further above, the user is then authenticated via Bcrypt and issued a `jwt` token based on their user id. 

The `jwt` token is then added to [`sessionStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage), a really neat Web API object you can use to store data.

So what can we do with this newly-issued `jwt` token? Well, you can do what I did ([following Sophie's lead](http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/) and add the following methods to your Application Controller, which all of your controllers will inherit.

```
class ApplicationController < ActionController::API
  # before_action :authenticate

  def logged_in?
    !!current_user?
  end

  def current_user?
    if auth_present?
      user = User.find(auth["user"])

      if user
        @current_user ||= user
      end
    end
  end

  def authenticate
    render json: { error: "unauthorized" }, status: 401 unless logged_in?
  end

  private

  def token 
    request.env["HTTP_AUTHORIZATION"].split(' ').last
  end

  def auth
    Auth.decode(token)
  end

  def auth_present?
    !!request.env.fetch("HTTP_AUTHORIZATION", "").split(' ').first
  end

end
```


You can authenticate all, some, or just a few of your controller actions by passing #authenticate to the appropriate controller callbacks (or action filters, if you will) like `before_action`.

Remember the Auth model I showed you earlier? Take a look at #auth in the code of above and see where the Auth model's .decode method comes into play. 

Pretty cool.

The final few pieces of this puzzle have to do with how to communicate to our API (which encodes and decodes the `jwt` token via its Auth model) which requests from our client are authorized.

In my app, the creation and deletion of cards from the user's favorites list require authentication. If we take a look in its FavoritesAPI, we can see just how to do this.

```
  static favorite(card) {
    const request = new Request('/favorites', {
      method: 'post',
      headers: new Headers({
        'Content-Type': 'application/json',
        'AUTHORIZATION': `Bearer ${sessionStorage.jwt}`,
        'Accept': 'application/json'
      }),
      body: JSON.stringify(card)
    });

    return fetch(request)
      .then(response => response)
      .catch(err => console.log(err))
  }
```

`favorite()` is a method on my FavoritesAPI class that is fired elsewhere in the app (when a user clicks an "Add to Favorites" button). It receives a particular card's data, instantiates a [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request), and passes that request object to `fetch`. `fetch` ships the request to my API's Favorites controller's `#create` (mapped to `/favorites` , via Rails conventions), and includes therewith a bit of very important information in the request header. If you take a close look, you'll see and `'AUTHORIZATION'` header containing an important-looking string: `Bearer ${sessionStorage.jwt}`. This string grabs the token (stored earlier) from sessionStorage and sends it along to the API, where it's decoded by #auth in the Application Controller. Noice again.

The last thing I'll point out here is the loginSucess() action I dispatched after adding the token to sessionStorage (see above for the code--look for all the arrows). This is one way you can mimic a session in an SPA using Redux.

Here's my the code for my sessions reducer (which receives the dispatched loginSuccess() action:

```
export default function sessionReducer(state = !!sessionStorage.jwt, action) {
  switch ( action.type ) {
    case types.LOG_IN_SUCCESS:
      return !!sessionStorage.jwt
    case types.LOG_OUT:
      return !!sessionStorage.jwt
    case types.CREATE_USER_SUCCESS:
      return !!sessionStorage.jwt
    default:
      return state;
  }
}
```

As you can see I coerced the value in `sessionStorage.jwt` to a boolean, and return that to state. I can access it from there via the props of any container in my React app, checking to see if the user accessng the app is logged in.

Noice thrice. 

That's about it from me. Again, I HIGHLY recommend taking a look at [Sophie's blog post](http://www.thegreatcodeadventure.com/jwt-authentication-with-react-redux/) on this topic if you want to add authentication to your SPA.

Cheers.
