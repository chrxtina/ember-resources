![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

# Ember CRUD

## Lesson Details
### Foundations
At this point, students have already learned how to:

- Configure the Ember Router to point to a new Template.
- Use an Ember View to handle events triggered within a routable template.
- Create nested view states and route to them appropriately.
- Set up resource routes (with their respective Ember Routes).
- Create Ember Components to represent UI elements and encapsulate related data and behavior.
- Use `ember-data` to set up Models representing business data.
- Link the `ember-data` data store to a JSON API through an Adapter.
- Set up a mock back-end using `ember-cli-mirage`

### Objectives
By the end of this lesson, students should be able to:

- Create Route actions and trigger them from Templates.
- Create Component actions and trigger them from Component Templates.
- Trigger Route actions from Component actions.
- Add behavior to Route actions to perform CRUD on the Route's model.
- Explain the meaning of the expression "actions up, data down".

### Setup
1. Fork and clone this repo.
2. Run `npm install && bower install`
3. Run `ember install ember-legacy-views`
4. Run `ember install ember-cli-mirage`, but _do not_ overwrite the `config.js` file. Additionally, delete the `scenarios` directory.

## The Shifting Landscape of Ember CRUD
Wow! We've really come a long way with Ember so far. We've learned all about the different parts of an Ember application: Templates, `Ember.View`, `Ember.Route`, `Ember.Router`, `Ember.Component`, `ember-data`, `DS.Model`... Now it's finally time to tie all of this together through one of the core functionalities of most applications, CRUD.

Before talking about CRUD, though, we should start by talking about 'actions'. 'Actions' are a special class of trigger-able events that are handled by the `Ember.ActionHandler` Ember Class. Like normal events, actions will 'bubble' - move from the leaf (i.e. Template) to the root (i.e. the 'application' Route), until they are met by a matching handler.

In Ember 1, actions were used inside the Controller to control Model CRUD. This made sense, since the Controller was responsible for managing all of the business data in our application; an action could be triggered in the Template, bubble up through the View, and reach a Controller, where it would cause the controller to manipulate Model data.

However, with the growing role of Components in Ember 2, a lot of the functionality of Controllers has been made redundant, and the current plan for Ember 2 calls for Controllers to be phased out completely; as a result, we need another place to put action handlers for performing Model CRUD. Fortunately, Components and Routes both incorporate `Ember.ActionHandler`, so we can instead set our action handlers inside either of those objects.

For simplicity's sake, we're going to put all handlers related to Model CRUD into the Route; any other handlers can be placed in either place.

Defining Action handlers in a Route is very easy. Simply open up the `route.js` file and make the following addition:
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  model: function(...){
    ...
  },
  actions: {
    create: function(){ ... },
    update: function(){ ... },
    destroy: function(){ ... }
    // ... etc
  }
});
```

Inside each of these actions, we can add code to manipulate that Route's model (which, conveniently enough, is provided within the Route). To trigger an action, you can add an `{{action ... }}` helper to an element (usually a button) - this will cause that element to launch the action whenever it executes its defaults behavior (in the case of a button, being clicked).

## Handling Actions in Practice
This has all been fairly abstract, so let's bring it down to earth by looking at how this works in an app. But not just any app...

![Charmander Used 'Ember'](./readme-assets/charmander.jpeg)

We're going to make a **Pokemon Directory (a.k.a. 'Pokedex')**, an application that keeps track of Pokemon that we've observed. This app will allows us to do the following CRUD operations:
- Add a new Pokemon to the directory.
- Edit an existing Pokemon.
- Remove a Pokemon from the directory.

As a first step for setting that up, let's give this application a structure for how it should handle actions; then, we can fill out the actions with whatever behavior that app requires.

Let's take a look at our application as it stands right now. It seems that we already have a Mirage test fixture in place, along with a Model to represent a Pokemon and an Adapter to handle the API transactions. Additionally, we have several Templates in place (some of which are already nested). As a result, we can click around and see a list of the Pokemon in our test fixture.

Let's add some action handlers to our Route by opening up `app/pokemon/route.js` and adding the following:
```javascript
actions: {
  createPokemon: function(){
    console.log('Route Action : createPokemon');
  },
  updatePokemon: function(){
    console.log('Route Action : updatePokemon');
  },
  destroyPokemon: function(){
    console.log('Route Action : destroyPokemon');
  }
}
```
Let's also add some HTML/Handlebars to the 'pokemon' Template:
```html
<button {{action 'createPokemon'}}>CREATE</button>
<button {{action 'updatePokemon'}}>UPDATE</button>
<button {{action 'destroyPokemon'}}>DESTROY</button>
```
We should now see three new buttons appear in the 'pokemon' view state. If we click one of these buttons, it will trigger the corresponding action in the Route.

As was mentioned, Routes are not the only things that can have actions; Components can have them too. Let's add some actions to the 'pokemon-snippet' Component:
```javascript
actions: {
  updatePokemon: function(){
    console.log('Component Action : updatePokemon');
  },
  destroyPokemon: function(){
    console.log('Component Action : updatePokemon');
  }
}
```
and a new button to the Template for 'pokemon-snippet':
```html
<strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
<button {{action 'updatePokemon'}}>EDIT</button>
<button {{action 'destroyPokemon'}}>EDIT</button>
<p> Generation: {{pokemon.generation}} </p>
<p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
```
Clicking these new buttons triggers their respective actions in the Component. Simple enough!

What if we want to trigger a Route action from within a Component? Because Components are essentially modular, this can only be accomplished by passing that action into the Component when the Component is created. Let's modify the 'pokemon' Template as follows:
```html
{{#each model as |eachPokemon|}}
  {{pokemon-snippet pokemon=eachPokemon routeUpdatePokemon='updatePokemon'}}
{{/each}}
```
This will make the `updatePokemon` and `destroyPokemon` Route actions available to the 'pokemon-snippet' Component, under the aliases of `routeUpdatePokemon` and `routeDestroyPokemon`, respectively. To actually trigger one of these actions from within the Component, we can call the method `sendAction`, passing in the name of the desired action as the first argument.
```javascript
actions: {
  updatePokemon: function(){
    console.log('Component Action : updatePokemon');
    this.sendAction('routeUpdatePokemon');
  }
},
```
Now our Component is capable of triggering Route actions!

### YOUR TURN : Handling Actions


## Non-CRUD Actions
There's no rule that actions need to be related to CRUD. Suppose we wanted to add a button to 'pokemon-snippet' that would toggle between hiding and showing the details (e.g. 'generation') of a given Pokemon. In our Component, let's create a new property, `isExpanded`, and a new action, `toggleExpanded`.
```javascript
export default Ember.Component.extend({
  tagName: 'li',
  twoTypes: Ember.computed('pokemon.typeOne', 'pokemon.typeTwo', function(){
    return this.get('pokemon.typeTwo') && this.get('pokemon.typeTwo') !== this.get('pokemon.typeOne');
  }),
  isExpanded: false,
  actions: {
    toggleExpanded: function(){
      this.toggleProperty('isExpanded');
    },
    updatePokemon: function(){
      console.log('Component Action : updatePokemon');
      this.sendAction('routeUpdatePokemon');
    },
    destroyPokemon: function(){
      console.log('Component Action : destroyPokemon');
      this.sendAction('routeDestroyPokemon');
    }
  }
});
```
Triggering the `toggleExpanded` action will allow us to toggle the value of the `isExpanded` property. Let's make some modifications to the Template for our Component.
```html
<strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
<button {{action 'toggleExpanded'}}>{{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}</button>
{{#if isExpanded}}
  <button {{action 'updatePokemon'}}>EDIT</button>
  <button {{action 'destroyPokemon'}}>DELETE</button>
  <p> Generation: {{pokemon.generation}} </p>
  <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
{{/if}}
```
Now every time the 'EXPAND'/'COLLAPSE' button is clicked, the content inside the `{{#if}}` will toggle between being visible and being hidden.

## CRUD : Destroying a Record
Now let's add some functionality behind those Route actions - any time the `destroyPokemon` Route action is triggered, we destroy a particular Pokemon. The way to destroy a given record from the data store is `<record>.destroyRecord()`. However, before we can destroy a record, we need to find it; we can do this using `<store>.findRecord(<type of record>, <id>)` (when invoked from a Route, the store is accessible at `this.store`). However, the `findRecord` method returns a _Promise_, so we'll need handle that Promise in the usual way.
```javascript
store.findRecord(<type of record>, <id>).then(function(<record>) {
  <record>.destroyRecord();
});
```
Inside the Route's `destroyPokemon` action, that code looks like this.
```javascript
export default Ember.Route.extend({
  model: function(){
    return this.store.findAll('pokemon');
  },
  actions: {
    // ...
    destroyPokemon: function(){
      console.log('Route Action : destroyPokemon');
      this.store.findRecord('pokemon', <id>).then(function(pokemon){
        pokemon.destroyRecord();
      });
    }
  }
});
```
As you can see, we're missing one critical piece of information : the `id` of the particular Pokemon we want to destroy. What we need to do is pass that information from the 'pokemon-snippet' Component (which has access to that particular record) back up to the Route. How can we do that?

As it turns out, the `sendAction` method can optionally accept additional arguments beyond the name of the action that it's triggering. Let's change the Components `destroyPokemon` action so that it passes in the `id` of the Pokemon it refers to.
```javascript
export default Ember.Component.extend({
  // ...
  actions: {
    // ...
    destroyPokemon: function(){
      console.log('Component Action : destroyPokemon');
      this.sendAction('routeDestroyPokemon', this.get('pokemon'));
    }
  }
});
```
Then, we change the Route action so that it can accept an argument, `id`.
```javascript
export default Ember.Route.extend({
  model: function(){
    return this.store.findAll('pokemon');
  },
  actions: {
    // ...
    destroyPokemon: function(pokemon){
      console.log('Route Action : destroyPokemon => destroying record with id ' + pokemon.get('id'));
      this.store.findRecord('pokemon', pokemon.get('id')).then(function(pokemon){
        pokemon.destroyRecord();
        console.log('record destroyed');
      });
    }
  }
});
```

As you can see, when we click the 'DELETE' button, the record for that Pokemon gets destroyed.

## CRUD : Adding a New Record
Adding a new Pokemon is a behavior tied to the _list_ of Pokemon instead of any particular Pokemon, so it would make the most sense to handle that behavior outside of the 'pokemon-snippet' Component. Suppose that we wanted this to be routable as well, e.g. `/pokemon/new`, so that the URL can be bookmarked. In that case, we would need to create a new Template using `ember g template pokemon/new`.

Let's populate that new Template with the following:
```html
<h5> Add a Pokemon to the directory! </h5>
{{input placeholder='National Pokemon Number'}}
{{input placeholder='Name'}}
{{input placeholder='Type One'}}
{{input placeholder='Type Two'}}
{{input placeholder='Generation'}}
<button {{action 'createPokemon'}}> Add a New Pokemon </button>

{{#link-to 'pokemon'}}Back{{/link-to}}
```

Additionally, let's create an 'index' Template on Pokemon, with a link pointing to this new 'pokemon/new' Template. We can do this by running `ember g template pokemon/index`, and filling the template with the following:

```html
{{#link-to 'pokemon.new'}}Add a new Pokemon{{/link-to}}
```

Last, but certainly not least, we need to (a) add an `{{outlet}}` to the 'pokemon' Template
```html

```
and (b) update the Ember Router to point to this new Template:
```javascript
Router.map(function() {
  this.route('pokemon', function(){
    this.route('new');
  });
});
```
As you can see, clicking the button labeled 'Add a New Pokemon' triggers the `createPokemon` action. How do we then translate that into performing CRUD on the model?

The method to add a new record to the data store is `<store>.createRecord(<type of record>, <new record data>).save()`, so let's leverage that inside our `createPokemon` Route action.
> .save() is necessary in order for the data store to give your new Pokemon an `id`

```javascript
export default Ember.Route.extend({
  model: function(){
    return this.store.findAll('pokemon');
  },
  actions: {
    createPokemon: function(){
      console.log('Route Action : createPokemon');
      var newPokemon = this.store.createRecord('pokemon', {
        nationalPokeNum: 201,
        name: 'Unown',
        typeOne: 'PSYCHIC',
        typeTwo: '',
        generation: 2
      });
      newPokemon.save().then(function(){
        console.log('record created');
      });
    },
    // ...
  }
});
```
> Get it? '[Unown](http://pokemondb.net/pokedex/unown)'?

As you can see, this will create a new Pokemon with the specified information every time we click the button. How do we instead tell it to use the information specified in the `<input>` fields?

This is a great use case for a Component - we could take data in our Template and bind it to the Component's own properties. However, we also specified that we want this to be routable, and Components are not yet routable. We could move the whole contents of `pokemon/new` into a Template, but an equally valid approach might be to store that data (while we still can) in a View.

Let's generate a new View for `pokemon/new` (`ember g view pokemon/new`) and fill it as follows:
```javascript
import Ember from 'ember';

export default Ember.View.extend({
  newPokemon: {
    nationalPokeNum: null,
    name: null,
    typeOne: null,
    typeTwo: null,
    generation: null
  }
});
```
This will give us a place to store our data. Now we have a problem, though - how do we sync the values of the `<input>` fields to our new View property? The answer is: **binding**. We can set `valueBinding` properties on each `<input>` element, persistently syncing them to properties on `view.newPokemon`.

```html
<h5> Add a Pokemon to the directory! </h5>
{{input placeholder='National Pokemon Number' valueBinding='view.newPokemon.nationalPokeNum' }}
{{input placeholder='Name' valueBinding='view.newPokemon.name' }}
{{input placeholder='Type One' valueBinding='view.newPokemon.typeOne' }}
{{input placeholder='Type Two' valueBinding='view.newPokemon.typeTwo' }}
{{input placeholder='Generation' valueBinding='view.newPokemon.generation' }}
<button {{action 'createPokemon' view.newPokemon}}> Add a New Pokemon </button>

{{#link-to 'pokemon'}}Back{{/link-to}}
```
We can verify that our binding is working correctly by temporarily adding `{{view.newPokemon.name}}` to our `pokemon/new` Template - if it syncs up with the name that's in the form, you're good to go.

The next step is to pass the View's `newPokemon` property into the `{{action}}` helper so that it can get shared with the Route's `createPokemon` action.
```html
<button {{action 'createPokemon' view.newPokemon}}> Add a New Pokemon </button>
```
Finally, we need to update our Route's `createPokemon` action so that it can accept `view.newPokemon` as an argument.

```javascript
export default Ember.Route.extend({
  model: function(){
    return this.store.findAll('pokemon');
  },
  actions: {
    createPokemon: function(newPokemonData){
      console.log('Route Action : createPokemon');
      // var newPokemon = this.store.createRecord('pokemon', {
      //   nationalPokeNum: 201,
      //   name: 'Unown',
      //   typeOne: 'PSYCHIC',
      //   typeTwo: '',
      //   generation: 2
      // });
      var newPokemon = this.store.createRecord('pokemon', newPokemonData);
      newPokemon.save().then(function(){
        console.log('record created');
      });
    },
    // ...
  }
});
```
Now we can create new Pokemon records by filling out our form!

## CRUD : Updating an Existing Record
Now that we've handled Create and Destroy, the last CRUD action to take care of is Update. There are several different ways to accomplish an update, depending on your UI, so let's start by making the following decision about how we want things to behave. **Keep in mind that these are just design decisions - your needs may differ, depending on your application.**
* Updating, like destroying, is specific to one particular record, so it might make sense to handle that behavior from within the 'pokemon-snippet' Component.
* Within the 'pokemon-snippet' Component, we should be able to toggle between 'editable' and 'not editable' states.
* This state change should be controlled by the 'EDIT' button that we already have.
* When the 'pokemon-snippet' Component is in its 'not editable' state, it should look the same as it currently does.
* When the 'pokemon-snippet' Component is in its 'editable' state, every data value should be replaced by an input box.
* The values of those input boxes should at least be set initially to the current values of those properties, and possibly should be bound permanently to said values.

Given these assumptions/decisions, let's see if we can work our way towards working Update behavior.

##### Updating, like destroying, is specific to one particular record, so it might make sense to handle that behavior from within the 'pokemon-snippet' Component.
##### Within the 'pokemon-snippet' Component, we should be able to toggle between 'editable' and 'not editable' states.
These indicate that we should have some sort of state property, attached to the 'pokemon-snippet' Component, which can be toggled by a button (implying that there must be a Component action to trigger the toggle). In that vein, let's add a new property and action to the 'pokemon-snippet' Component : `isEditable` and `toggleEditable`, respectively.
```javascript
export default Ember.Component.extend({
  tagName: 'li',
  twoTypes: Ember.computed('pokemon.typeOne', 'pokemon.typeTwo', function(){
    return this.get('pokemon.typeTwo') && this.get('pokemon.typeTwo') !== this.get('pokemon.typeOne');
  }),
  isExpanded: false,
  isEditable: false,
  actions: {
    toggleExpanded: function(){
      this.toggleProperty('isExpanded');
    },
    toggleEditable: function(){
      this.toggleProperty('isEditable');
    },
    updatePokemon: function(){
      console.log('Component Action : updatePokemon');
      this.sendAction('routeUpdatePokemon');
    },
    destroyPokemon: function(){
      console.log('Component Action : destroyPokemon');
      this.sendAction('routeDestroyPokemon', this.get('pokemon'));
    }
  }
});
```

##### This state change should be controlled by the 'EDIT' button that we already have.
Straightforward enough. Let's change the EDIT button so that it triggers the new `toggleEditable` action.
```html
<strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
<button {{action 'toggleExpanded'}}>
  {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
</button>
{{#if isExpanded}}
  <button {{action 'toggleEditable'}}>EDIT</button>
  <button {{action 'destroyPokemon'}}>DELETE</button>
  <p> Generation: {{pokemon.generation}} </p>
  <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
{{/if}}
```

> Although this wasn't mentioned as a requirement, it might be a little weird if we were to set `isEditable` to true, and then for it to remain true after we've collapsed and then reopened the Component. Let's change our `toggleExpanded` action so that it sets `isEditable` to false every time the Component is collapsed.
```javascript
export default Ember.Component.extend({
  // ...
  isExpanded: false,
  isEditable: false,
  actions: {
    toggleExpanded: function(){
      this.toggleProperty('isExpanded');
      if (!this.get('isExpanded')) {
        this.set('isEditable', false);
      }
    },
    // ...
  }
});
```

##### When the 'pokemon-snippet' Component is in its 'not editable' state, it should look the same as it currently does.
##### When the 'pokemon-snippet' Component is in its 'editable' state, every data value should be replaced by an input box.

There are probably a couple of ways you could do this, but what would probably by easiest is simple to have two separate sections of the Component's Template, switched by an `{{#if}}` or `{{#unless}}` helper. Since the normal state should show up when the Component is _not_ editable, it would probably make sense to use `{{#unless}}`.

```html
{{#unless isEditable}} {{!-- Non-editable Version --}}
  <strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  {{#if isExpanded}}
    <button {{action 'toggleEditable'}}>EDIT</button>
    <button {{action 'destroyPokemon'}}>DELETE</button>
    <p> Generation: {{pokemon.generation}} </p>
    <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
  {{/if}}
{{/unless}}
```
To handle the opposite case, we can add an `{{else}}` helper.
```html
{{#unless isEditable}} {{!-- Non-editable Version --}}
  <strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  {{#if isExpanded}}
    <button {{action 'toggleEditable'}}>EDIT</button>
    <button {{action 'destroyPokemon'}}>DELETE</button>
    <p> Generation: {{pokemon.generation}} </p>
    <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
  {{/if}}
{{else}}  {{!-- Editable Version --}}
  <strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  {{#if isExpanded}}
    <button {{action 'toggleEditable'}}>EDIT</button>
    <button {{action 'destroyPokemon'}}>DELETE</button>
    <p> Generation: {{pokemon.generation}} </p>
    <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
  {{/if}}
{{/unless}}
```
Now we can swap all of the normal HTML-escaped values (`{{ someValue }}`) for `{{input}}` helpers, so that they can generate new `<input>` elements.
```html
{{#unless isEditable}} {{!-- Non-editable Version --}}
  <strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  {{#if isExpanded}}
    <button {{action 'toggleEditable'}}>EDIT</button>
    <button {{action 'destroyPokemon'}}>DELETE</button>
    <p> Generation: {{pokemon.generation}} </p>
    <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
  {{/if}}
{{else}}  {{!-- Editable Version --}}
  <strong>
    #{{input}} : {{input}}
  </strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  <button {{action 'toggleEditable'}}>EDIT</button>
  <button {{action 'destroyPokemon'}}>DELETE</button>
  <p> Generation: {{input}} </p>
  <p> Type: {{input}} / {{input}} </p>
{{/unless}}
```
Let's also change the 'EDIT' button to say 'CONFIRM EDIT' in the isEditable case.

##### The values of those input boxes should at least be set initially to the current values of those properties, and possibly should be bound permanently to said values.
The `{{input}}` helper allows for binding the value of an input box to another variable, so for simplicity's sake, let's just opt for that solution and bind all of the input boxes to their respective properties in the Component.
```html
{{#unless isEditable}} {{!-- Non-editable Version --}}
  <strong>#{{pokemon.nationalPokeNum}} : {{pokemon.name}}</strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  {{#if isExpanded}}
    <button {{action 'toggleEditable'}}>EDIT</button>
    <button {{action 'destroyPokemon'}}>DELETE</button>
    <p> Generation: {{pokemon.generation}} </p>
    <p> Type: {{pokemon.typeOne}} {{#if twoTypes}}/ {{pokemon.typeTwo}}{{/if}} </p>
  {{/if}}
{{else}}  {{!-- Editable Version --}}
  <strong>
    #{{input valueBinding='pokemon.nationalPokeNum'}} : {{input valueBinding='pokemon.name'}}
  </strong>
  <button {{action 'toggleExpanded'}}>
    {{#unless isExpanded}}EXPAND{{else}}COLLAPSE{{/unless}}
  </button>
  <button {{action 'toggleEditable'}}>CONFIRM EDIT</button>
  <button {{action 'destroyPokemon'}}>DELETE</button>
  <p> Generation: {{input valueBinding='pokemon.generation'}} </p>
  <p> Type: {{input valueBinding='pokemon.typeOne'}} / {{input valueBinding='pokemon.typeTwo'}} </p>
{{/unless}}
```

##### Cleanup
I guess we didn't need those `updatePokemon` actions after all - let's remove them. Let's also get rid of the three buttons we added to the 'pokemon' Template, since those were just for testing. Once we're done with those things we should be good to go, since the app can now Create, Update, and Destroy!


## Additional Resources
- [Ember API : Ember.ActionHandler](http://emberjs.com/api/classes/Ember.ActionHandler.html)
- [Ember API : DS.store](http://emberjs.com/api/data/classes/DS.Store.html)
