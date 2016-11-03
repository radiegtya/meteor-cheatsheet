# Meteor Cheatsheet
Meteor Cheat Sheet and Best Practices. This cheatsheet are suitable to used for Meteor v.1.2.1 or lower, but you can also used this for the latest meteor version with some little improvement. 

## 1. Folder Structure
Meteor 1.2.1 and lower have strictly located folder structure, so follow this for the best practice.

```
your-project-root/
  client/             # Client only folder
     components/      # For html/js files which act as reusable components
     scenes/          # For html/js files which act as menu/scene/route
     
  lib/                # Both Accessible folder for Client & Server Code
     collections/     # Collections folder
     router.js        # Router js file
     
  server/             # Server only folder          
```

Let's take a look at some example. This one is an example of Blog app folder structure

```
your-project-root/
  client/                    # Client only folder
     scenes/                 # For html/js files which act as menu/scene/route
        layout/              # For layouting, and your first file to be called/rendered
          main/              # The layout name called "main", you can also create another layout with other name to switch
            index.html       # Your main container on layout html file
            index.js         # Your main container on layout js file
            navbar.html      # Your navigation bar to be attached at (layout/main/index.html)
        post/
          index.html         # List of posts html file        
          index.js           # List of posts js file
          view.html          # Detail of post html file
          view.js            # Detail of post js file
        comment/
          index.html         # List of comment in a post html file
          index.js           # List of comment in a post js file
         
     components/             # For html/js files which act as reusable components
          Readmore.js        # Let u easily create Long text to short text 
          ImageGallery.js    # Easily show image list of a post js file
          ImageGallery.html  # Easily show image list of a post html file                    
     
  lib/                       # Both Accessible folder for Client & Server Code
     collections/            # Collections folder
        Post.js              # Mongo Collection from "post"
        Comment              # Mongo Collection from "comment"
     router.js               # Router js file, to define router
     
  server/                    # Server only folder 
      PostServer.js          # Server Publication and Method for "post"
      CommentServer.js       # Server Publication and Method for "comment"      
```

## 2. Blaze Component
Rather than using original Blaze Template, it will be better to use Blaze Component (this is inspired by React). 

### 2.1. Blaze Component Quick Start
a. Installation
```
meteor remove templating
meteor add peerlibrary:blaze-components
```
b. Naming Convention

In order to make great app, you must follow this naming convention and folder structure
To decide best template naming convention, follow these steps:
- for example you have collection name called "post" which is instantiated to a variable called "Post"

```
Post = new Meteor.Collection('post');
```
- and have routing like this
```
/post
/post/:_id
/post/update/:_id
/post/insert
```
- so you need to have template name like this
```
/post                      =>         PostIndex
/post/:_id                 =>         PostView
/post/update/:_id          =>         PostUpdate
/post/insert               =>         PostInsert
```

c. Template inclusion

Template inclusion is the way to instantiate your template in your html by referencing from your js file.
Now let's try to make template inclusion

create js file "/client/scenes/post/index.js"
```
class PostIndex extends BlazeComponent{  
  
}
PostIndex.register('PostIndex');
```

create html file "/client/scenes/post/index.html"
```
<template name="PostIndex">
  Hello World
</template>
```

now let's make the inclusion wherever you want. 
(Note: in real world app, the one thing that need inclusion only from layout with dynamic inclusion from routing. Except component inclusion.)

```
{{> PostIndex }}
```
this way will print "Hello World" on your screen.

d. Template helpers

edit js file "/client/scenes/post/index.js"
```javascript
class PostIndex extends BlazeComponent{  
  
   //this one called Template#helpers which can be passed to the html
   myName(){
    return "Ega Radiegtya";
   }
  
}
PostIndex.register('PostIndex');
```

edit html file "/client/scenes/post/index.html"
```
<template name="PostIndex">
  Hello {{myName}}
</template>
```
use {{}} to call helpers, then your screen will print "Hello Ega Radiegtya"

e. Template events

for example adding events code in "/client/scenes/post/index.js"

```javascript
class PostIndex extends BlazeComponent{  
  ...
  
  post(){
    return Post.findOne({title: "Jordan the Bad Boy"});
  }
  
  handleLike(e){
    e.preventDefault();
    console.log("Like btn pressed");
  }
  
}
...
```

html file "/client/scenes/post/index.html"

```
<template name="PostIndex">
  ...
  {{#with post}}
    {{title}}
    <button onClick="{{handleLike}}">Like this post</button>
   {{/with}} 
</template>
```
there are another event like "onKeyup", "onKeydown", etc.

f. Template data

example get current data inside each tag, or with tag, or data props in js helper
```javascript
getCurrentDataAndModify(){
  //don't confuse! This is ES6 syntax similar to var firstData = this.currentData; var secondData = this.currentData();
  const {firstData, secondData} = this.currentData(); 
  return firstData + secondData;
} 
```

example get current data in events

```javascript
handleLike(e){
  e.preventDefault();
  const {_id, currentLike} = this.currentData();
  
  Post.update(_id, {$set: {
    currentLike: currentLike + 1,
  }});
}
```

example get current data inside each tag, or with tag, or data props in js helper
```javascript
getParentDataAndModify(){
  const {firstData, secondData} = this.data();
  return firstData + " - " + secondData;
} 
```

For more info about using Blaze Component, follow this link:
https://github.com/peerlibrary/meteor-blaze-components


## 3. Flow Router

Use this router package for production ready routing.
```
meteor add kadira:flow-router
```
Complete Doc COMING SOON. For now refer to this link https://github.com/kadirahq/flow-router

## 4. MongoDB Schema with Collection2

Use this package to scheming your mongoDB, field validation, etc.
```
meteor add aldeed:collection2

and this will automatically add:
meteor add aldeed:simple-schema
```
Complete Doc COMING SOON. For now refer to this link https://github.com/aldeed/meteor-collection2

## 5. Form

Use this package to make cool form with validation inside

```
meteor add aldeed:autoform
```

Complete Doc COMING SOON. For now refer to this link https://github.com/aldeed/meteor-autoform

## 6. Publish & Subscribe Best Practice with Publish Composite

```
meteor add reywood:publish-composite
```

Server Example:
```javascript
Meteor.publishComposite('post', function(selector, options) {
    return {
        find: function() {
            // Find posts with selector and options passed from client
            return Post.find(selector, options);
        },
        children: [
            find: function(collection) {
                // Find comment relation (HAS MANY COMMENT)
                return Collection.find({postId: collection._id});
            }
        ]
    }
});
```

Client Example, for template level subscriptions:

on template component subscribe needed data:
```javascript

PostIndex extends BlazeComponent{

  onCreated(){
    this.autorun(()=>{
      this.subscribe('post', this.selector(), this.options());
    });
  }
  
  //your query goes here
  selector(){
    return {title: {$regex: "abc", $options: "i"}};
  }
  
  //your options like limit, and sort goes here
  options(){
    return {limit: 10, sort: {title: -1}};
  }

}

```

Complete Doc COMING SOON. For now refer to this link https://atmospherejs.com/reywood/publish-composite

## 7. Reactive Variable

Reactive variable is a variable which changed automatically when emitted or there is an event that trigger it to change. Never use Meteor.session!!! Use Meteor Reactive Var insted.

```
meteor add reactive-var
```

simple example:
```javascript
//init limit
let limit = new ReactiveVar(5);

//set limit value reactively
limit.set(15);

//get limit value reactively on template lifeCycle like onCreated, onRendered etc
this.autorun(()=>{
  //pass the value to this.limit inside Tracker.autorun
  this.limit = limit.get();
});

//get limit value reactively on template helper
iAmTemplateHelperToGetLimit(){
  return limit.get(); //you don't need to use Tracker.autorun in template helpers
}

```

Complete Doc COMING SOON. For now refer to this link https://docs.meteor.com/api/reactive-var.html


## 8. Cybermantra Framework

List of packages that make your day.

### 8.1. Cybermantra DataProvider
CybermantraDataProvider is a class to Store any reactive data to be distributed to another cybermantra plugins.

##### Quick Start

- Installation

```
meteor add cybermantra:data-provider
```

- instantiate

```
//instantiate without collection
const dataProvider = CybermantraDataProvider();

//or you can instantiate with collection
#const dataProvider = CybermantraDataProvider(MyCollection);
```

- getter

```
CybermantraDataProvider()#options.get()

//ex:
const dataProvider = CybermantraDataProvider();
dataProvider.selector.get();
```

- setter

```
CybermantraDataProvider()#options.set(val)

//ex:
const dataProvider = CybermantraDataProvider();
dataProvider.selector.set({title: "blah"});
```

##### CybermantraDataProvider Available Options
CybermantraDataProvider have some options which can be stored and used by another packages, here is the list:

- collection: Collection Obj
- selector: Reactive Obj
- options: Reactive Obj
- options.sort: Obj
- options.limit: Number
- options.skip: Number
- count: Reactive Number
- currPage: Reactive Number
