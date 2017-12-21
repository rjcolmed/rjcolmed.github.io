---
layout: post
title:      "Using Semantic UI React"
date:       2017-12-21 00:07:19 +0000
permalink:  using_semantic_ui_react
---



For my recently completed final project, a React/Redux client powered by a Rails API, I continued my tradition of using Semantic UI to make things look nice.

Why use the same frontend framework, for 3 projects, no less?

First, I like the way it looks. More so than Bootstrap, its nearest, most comparable alternative.

More importantly, though, it's easy to use. Probably not easier than Bootstrap, but most certainly on the same level.

Let's look at a few examples from my project. The following is a section from a file meant to represent the show page of a Netrunner card.

```
import React from 'react';
import { connect } from 'react-redux';
import {  bindActionCreators } from 'redux';
import * as actions from '../actions/favorites_actions'
import { Card, Image, Button } from 'semantic-ui-react'; 

class CardShow extends React.Component {

  handleOnClick = event => {
    event.preventDefault();
    this.props.actions.addToFavorites(this.props.card)
    .then(this.props.history.push('/cards', { favoritedCard: this.props.card }));
  }

  removeFromFavorites = event => {
    event.preventDefault();

    this.props.actions.removeFromFavorites(this.props.card)
    .then(this.props.history.push('/cards', { unfavoritedCard: this.props.card }));
  }

  render() {
    const { favorites, card, logged_in } = this.props 

    return (
      <Card>
        <Image src={ card.image_url }/>
        <Card.Content>
          <Card.Header>{ card.title }</Card.Header>
          <Card.Meta>{ card.flavor }</Card.Meta>
          <Card.Description>{ card.text }</Card.Description>
        </Card.Content>
        { logged_in &&
        <div>
          { favorites.find(favorite => favorite.id === card.id) ?
            <Button 
              attached="bottom" 
              onClick={ this.removeFromFavorites }
            >Remove from Favorites
            </Button> :
            <Button
              attached="bottom"
              onClick={ this.handleOnClick }
              >Add to Favorites
            </Button>
          }
        </div>
        }
      </Card>
    );
  }
}

```

As you can see, Semantic UI React made it pretty easy for me to quickly, cleanly add style to my card show page. All I had to do was import the components I needed  and voila.

You also might notice that the main component is called `<Card />`. That's native to Semantic UI, so it worked out pretty well for me.

Check out the the [card page](https://react.semantic-ui.com/views/card) on Semantic's website for more info on this cool component.

 [Here's the final product](https://imgur.com/a/hlZsW)
 
 Anyway, I highly recommend using Semantic UI, particulary for non-React projects. Why? One, from a quick overview of both, they're not much different in React.  Two, as the name suggests. Semantic UI is semantic. The naming conventions it uses for its classes are super easy to understand and use. 
 
 One example of this before I go.
 
 Here's an example of two similar style classes for buttons
 
 Bootstrap:
 
 ```
 <button class="btn btn-primary">Primary</button>
 ```

Semantic UI

```
<button class="ui primary button">Primary</button>
```

See how Semantic is more...semantic? 
