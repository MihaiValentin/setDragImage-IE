# setDragImage support for Internet Explorer 10+ Drag & Drop

If you plan to use HTML5 Drag & Drop in your app, you will find out the hard way that Internet Explorer does not support the `setDragImage` method, which on Firefox / Chrome shows an arbitrary image next to the cursor while dragging.

This polyfill provides the same functionality for Internet Explorer. It is less than 800 bytes compressed and gziped.

## Why should you use it?

Generally the missing support for a drag image does not cause issues. However, there are cases when you want to give real feedback to the user related to what they are actually dragging. Let's say that if they drag one item, you should show an icon of a file, and when they drag multiple items you should show an icon of multiple files (and eventually a number). As Internet Explorer just shows as the drag element the element that is currently being dragged, the visual feedback can be confusing.

## Demo

Check the demos in the `demo` folder

## How to use it?

* include `setdragimage-ie.js` in your project
* outside the `dragstart` event, (preferably when your application loads), call `setDragImageIEPreload(img)` to preload the image which you will later use.
    * Preloading ensures the image will be ready to be used when the first drag will happen
    * `img` is a HTML image (either an image element or a Image object)
* inside the `dragstart` event, just use `ev.dataTransfer.setDragImage(img, x, y)` as you'd do it for Chrome / Firefox
* start dragging and you'll also get the drag image on Internet Explorer as well

## Installation via NPM
```npm install setdragimage-ie --save```

## How does this actually work?

I noticed that if you make a change to the element's style (adding a class that changes appearance) inside the `dragstart` event and then removing it immediately in a `setTimeout`, Internet Explorer will make a bitmap copy of the modified element and will use it for dragging.
So, what this library actually does is implement the `setDragImage` method that changes the target's element style by adding a class that includes the image that you want to appear while dragging, and then removes it. In this way, the browser displays the temporary style of the element as the drag image.

## Limitations

Due to the technical details described above, this polyfill has the following limitations / issues:

* there will be a very slight flicker when the drag will start
* you cannot specify the mouse offsets (the second and third parameter for `setDragImage`
    * the reason is that when IE makes the bitmap copy, he does not understand transparency and uses white. Any attempt to position the background image will result in a lot of white near the image, creating a very ugly effect
* the code is not in strict mode
    * the reason is that `setDragImage` is ran on the `DataTransfer` property of the event, so inside it the `this` is not the element or the event. To overcome this, I've used `caller` so I walk through the functions that called `setDragImage` and when I meet one that has the first parameter of type `DragEvent` (the `dragstart` event), I get it and then get it's target. Another idea would have been to add the fourth parameter so you can send the element directly, but it is not feasible, since new methods may be added on `setDragImage` in the future and it is not forward-compatible.
