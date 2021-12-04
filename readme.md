# simple-popup-box
This is an example of a basic popup box which opens and closes on a button click. The contents are static here but the code can be enhanced to use dynamic messages. The sections below will outline a walkthrough of the process used to create this index.html. Feel free to also check out:
* Our [YouTube Video](https://www.youtube.com/watch?v=EGNRVStdtEw), where we code this in real time with music.
* Our [Dev Page](https://bytethisstore.com/articles/pg/simple-popup), where we cover this topic in more detail.

## Demo the Live Example
The live example can be viewed in action on [this page](https://htmlpreview.github.io/?https://github.com/ByteThisStore/simple-popup-box/blob/main/index.html).

## Popup Box Requirements
We're going to create a fairly simple & straightforward popup box, but before getting into the code, let's explicitly outline the desired behavior of the popup box:
* It should be hidden by default.
* We should be able to trigger it to open and close via JavaScript.
* Calling open should do nothing if the box is already open.
* Calling close should do nothing if the box is not already open.
* When the popup is open, we should display an overlay so the user can't click anywhere on the document.
* The popup should appear in the center of the user's view, regardless of their scroll position.
To summarize the points above, we want the popup box to be in a well defined state at all times. Furthermore, the client (the code using the popup box) shouldn't have to worry about checking if the box is open already before opening it or checking if its closed already before closing it.

## HTML Template Element
There are many ways we can create a popup box, in terms of specifying where to put the contents and how to activate the popup. In this implementation, we're going to use a **template** element to encapsulate the contents of the popup box. This is a special type of element; it's contents are not rendered immediately. We can define the elements which will appear in the template in the same way we can for any normal element: declare the elements within the **\<template\>** element's open and close tag. For example:
```html
<template id='popup-template'>
    <p>Hello World</p>
</template>
```

In order to actually render the contents of a template, we'll need to.
1. Use JavaScript to query the template and assign its reference to a variable; we can attach an **id** to the template to make that easy.
1. Perform a deep **clone** on the template reference and store that in another variable. At this point, we'll have an element reference, similar to the case where we would create an element reference using JavaScript instead of cloning, and can manipulate it if we need to.
1. Attach the cloned element to the place in the document where we want it to appear.
In that sense, it doesn't necessarily matter where we put the template initially; we can clone and place it where we need it to be when the time comes. The code below shows an example of cloning a template node:
```javascript
//clone the template object so we can use it
const template = document.getElementById("popup-template");
const clone = template.content.cloneNode(true); //we need the "true" flag for this to work properly

//for simplicity, we'll append directly to the body; we could append elsewhere as well
document.body.appendChild(clone);
```
In the following sections, we'll outline where and when to use that code in order to render the popup.

## Opening and Closing the Popup with JavaScript
Now that we have the template in place, we can write the logic to show and hide the popup. There are many ways we can write such logic; in this example we'll do so using a class with static methods. In the **open** method, we'll use code similar to the code above to clone the template and attach it to the document. In the **close** method, we'll remove the element we've previously attached. A skeleton version of the implementation will look something like this:
```javascript
class Popup {

    static open() {
        //clone the template object so we can use it
        const template = document.getElementById("popup-template");
        const clone = template.content.cloneNode(true);

        document.body.appendChild(clone);

        //listen to the "ok button" click to close
        const okButton = document.getElementById("popup-ok");
        okButton.addEventListener("click", () => {
            Popup.close();
        });
    }

    static close() {
        //remove previously placed element
        const openedPopup = document.getElementById("popup-container");
        document.body.removeChild(openedPopup);
        console.log("Popup is now closed!");
    }
}
```
In the example above, we're doing some basic logic to clone the template node, place it in, and upon close, remove that cloned node. Note that we haven't put logic in for checking if the popup is already open or closed, that will be covered in the sections below.

## Structuring the Popup Element
In order to visually render the popup correctly, even when the JavaScript is in place, we will need to ensure the html structure and css rules are applied appropriately:
* The HTML structure needs to define a backdrop / background to prevent the user from interacting with the form, then the popup itself over top of that.
* The CSS structure needs to position the elements properly, size the correctly, and apply the correct colors and fonts.
For the HTML structure, we can create a div which behaves like a container for the element, then place the popup box itself inside of it. For example:
```html
<!-- Container for popup + background -->
<div id="popup-container">
    <!-- Box for actual popup -->
    <div id="popup-box">
        <h2>Popup!</h2>
        <p>This is a popup box!</p>
        <p id="like-p">
            Don't forget to follow us on social media!
        </p>
        <button id="popup-ok">Ok!</button>
    </div>
</div>
```
The intent for the elements above is: the container will cover the entire screen and prevent the user from interacting with the rest of the page. The popup box will sit on top in the center of the screeon.

## Styling the Popup Element
Once the HTML structure is in place, we will need to apply CSS styles so it positions itself properly. To make the overlay prevent the user from interacting with the rest of the page, we'll set it's **z-index** property to 1 (or some arbitrarily high value) and set the popup's z-index to something slightly greater, 2 in this case. We'll also position the overlay to fill the entire screen and have the popup appear in the exact center of the screen.
```css
#popup-container {
    /* Put in front of all other elements to prevent interaction */
    z-index: 1;

    /* Absolute position lets us cover the screen regardless of scroll position */
    position: absolute;

    /* Make it cover the entire screen using left+top plus width+height */
    left: 0;
    top: 0;
    width: 100vw;
    height: 100vh;

    /* Prevent scrollbar during animation */
    overflow: hidden;

    /* Transparent background color (overlay) */
    background-color: rgba(0, 0, 0, 0.5);
}

#popup-box {
    z-index: 2;
    position: absolute;

    /* Same positioning strategy, but should only take up center part of the screen */
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    background-color: white;
    border: 2px solid black;
    padding: 20px;
}
```
We can slightly expand upon this CSS to have the popup box make an animated entrance. We'll create a **keyframes** rule to specificy the behavior of the animation at the start and end states. The end state should match the state we want the element to be in when the animation ends. In the keyframes below, we are declaring that the popup box should appear from the bottom and work its way up to the middle.
```css
/* animation specific to modal popup, start low end high */
@keyframes slide-up {
    0% {
        top: 100%;
    }

    100% {
        top: 50%;
    }
}

/* add declaration to the popup element to use this animation */
#popup-box {
    /* ..existing css..*/

    /* animation should use our "slide-up" and take 1/5 of a second */
    animation: slide-up 0.2s ease-in;
}
```

## Popups with Dynamic Contents
In this example, we've been using a popup with static contents, but a common requirement for a popup box is to have dynamic contents: contents which can be changed using JavaScript. Coding this is outside of the scope of this example, but it is worth discussing. We could modify our approach to accomodate this by:
1. Put a container element, such as a span or paragraph, in the template and give it an id such as "contents-container".
1. Enhance the JavaScript class to take a "contents" parameter in the "open" method where we can specify our contents. If nothing is provided, we can use some default contents.
1. In the "open" method, if custom contents are fed in, select the contents-container and change its inner text or html to the contents provided.
We could also take a similar approach to allow for specifying which buttons we should display. Two common approaches to this are:
* Give a pre-specified set of options for the user, such as { Ok, Cancel }, { Confirm, Cancel }, { Ok }, and let the consumer specify which grouping.
* Let the user fully specify which buttons to include
* Take a hybrid approach between the two previous choices.
In any case, if we have more than one button, the Popup class will need some way of letting the consumer know which button was selected. This can be done with an event emitter, Promises, or with other strategies.
