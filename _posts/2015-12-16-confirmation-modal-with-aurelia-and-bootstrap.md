---
layout: post
title: Confirmation Modal With Aurelia And Bootstrap
category: Aurelia
---

I have lots of (different) items in my view that can be deleted. And I want a confirmation dialog to pop up when I press the delete button. In this post I'll show you how to implement this functionality in [Aurelia](http://aurelia.io/) using Bootstrap for styling. 

One thing that I like about Aurelia is how easy it is to write maintainable components. So, I will create a `delete` custom element that will trigger our confirmation dialog. Because it's a generic element, I want it to be easily "attached" to any item that needs to be deleted. And I want to set the confirmation text and the action to be execute on "yes" .

## The Confirmation Dialog

But first things first. We need to create the modal that will be invoked by Delete. Luckily, there is a plugin for Aurelia that does that. Let's install it.

```
jspm install aurelia-dialog
``` 

In order to let aurelia know about it, we need to add this bit in the configuration function

```javascript

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
	.plugin("aurelia-dialog")
	;

  aurelia.start().then(() => aurelia.setRoot());
}

```

Now we need to create the modal element itself. The plugin comes with a 'Prompt' example, but we won't be using it. So let's create the confirmation modal. First the view model (I'm using Typescript)

```javascript

import {autoinject} from "aurelia-framework";
import {DialogController} from 'aurelia-dialog';

@autoinject
export class Confirm {

    constructor(public controller: DialogController) {
      
    }
    message ="";
    activate(data) {
        this.message = data;
    }
}

```

Now, the view. We can use most of the Bootstrap styling without the js functionality.

```html

<template>
    <div tabindex="-1" role="dialog">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" click.trigger="controller.cancel()" class="close" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title">Confirmation</h4>
                </div>
                <div class="modal-body">
                   ${message}?
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-default" click.trigger="controller.cancel()">No!</button>
                    <button type="button" class="btn btn-danger" click.trigger="controller.ok()">Yes</button>
                </div>
            </div><!-- /.modal-content  -->
        </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->

</template>


```

Note how the dialog div is missing the usual "modal fade" classes. It's because aurelia will take care of creating, displaying and hiding the modal. The rest of the code is pretty much self explaining. We control the outcome of the dialog invoking the controller (DialogController) `ok()` or `cancel()` methods. Both will trigger the closing of the dialog and will set up the result that we can use further. 

One thing worth mentioning is you can pass a model to the `ok` method. For example, if the dialog was for a form, whose fields were bound to a model, I could pass that model to be returned like `controller.ok(myModel)` . You can read more about this on the [plugin page](https://github.com/aurelia/dialog#using-the-plugin) .
   
Now, we won't be using this element directly, we'll be opening the dialog through `DialogService`. Or more precisely the `Delete` element will do that.

## The Delete element

This is what will be actually using in our templates. Let's see its definition:

```javascript

import {autoinject, bindable} from "aurelia-framework";
import {DialogService} from "aurelia-dialog";
import {Confirm} from "./confirm";

@autoinject
export class Delete {
   
    @bindable
    action=()=>{};

    @bindable
    msg = "Are you sure";

    constructor(public dlg:DialogService) {
  
    }

    do() {
        this.dlg.open({
            viewModel: Confirm
            , model: this.msg
        }).then(result => {
            if (result.wasCancelled) return;
            this.action();
        });
    }
}

``` 

```html
<template>
    <button class="btn btn-danger" click.trigger="do()">Delete</button>
</template>

```

Nothing complex here. The element will accept the values of the message and of the action to be taken on "Yes". Then when the button is clicked, the `do()` method invokes the DialogService's `open()` method, passing the type of the confirmation's view model and the model (in this case a simple string) used by it. The method returns a Promise where we can set up what happens after the dialog is closed.

The 'result' argument has 2 fields: `wasCancelled` which is obvious what it does and `output` which is the modified model that is returned by the dialog (if any). In this scenario we don't need it. 

## Actual usage

Now we can use our `Delete` element where we need it. As an example, let's say I have a list of names

```javascript

    data=['John','Mary','David'];
    
    delete(name:string){
       
       this.data.splice(this.data.indexOf(name), 1);            
        
    }

    deleteOther(){
        console.log("something deleted");
    }

```

```html
<require from="./delete"></require>

<delete msg="Are you sure" action.call="deleteOther()"></delete>

<ul>
 <li repeat.for="item of data">
     ${item}
       <delete msg="Are you sure you want to delete ${item}" action.call="$parent.delete(item)"></delete>   
 </li>
</ul>
   
```` 

Note the binding to a function syntax `action.call` . This tells Aurelia to execute the specified expression every time `action()` is invoked. 

And this is it! As you've seen, it's quite simple to create your own custom dialog with Aurelia while using Bootstrap CSS. Before I knew about the dialog plugin, I had my own custom element which used Bootstrap js and jQuery and while it's not complicated, it's not as clean and smooth as it is with 'aurelia-dialog'. This means that even if there is no plugin to offer the functionality you need, it's still trivial to use existing Boostrap or JQuery plugins.    