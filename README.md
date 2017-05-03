# Illustrator Script Development (Beginner)
## What is an Illustrator Script?

If you're working with Adobe Illustrator on a daily basis, you might have faced a seemingly easy problem that Illustrator does not provide a tool for you to solve. It might even be a task that you have to do very often and every time you have to do the little workaround that you got accustomed to. Something like:
- setting your artboard dimensions to full pixels and snap them to pixel positions
- flipping a row of objects from horizontal to vertical or vice versa
- dividing a text frame with multiple lines into separate text objects for each line

You can add missing functionality by writing a piece of javascript code, then execute it in Illustrator and your desired outcome is done with one click. Creating a script is pretty straight forward: Create an empty `myScript.js` file in a folder of your choice, then describe what you want to get done. For starts, we will write a minimal script that writes the name of your active document to the console.
1. In your `myScript.js` file write `$.writeln(app.activeDocument.name);`
2. In Illustrator, go to `File > Scripts > Other Scripts` and then navigate to where you saved the file and open it.
3. Adobe ExtendScript Toolkit should pop up in your task bar
4. Inside ExtendScript Toolkit, press `ctrl + j` or use menu bar: `Window > Javascript Console` to bring up the console
5. In the console, you should see a line telling you:  _nameOfYourDocument_

Nice, now you know the basics of how to write and execute an Illustrator script and how to investigate information about your document using the console. Also notice that we have a global `app` variable available for us at all times. It holds all information about the current state of Illustrator with all its open documents and the graphical elements living within them.

#### Step-by-step script reference
There's an accompanying git repository that has all the ready-to-use scripts we will create here. You can cross-check each individual step with your own script. In the terminal, browse to your project folder and type  `git clone https://github.com/s-haensch/illustrator-scripting-beginner.git`.  

The steps will be indicated like so:  
_script reference:_ `git checkout [name-of-step]`  

To see the result of the part above, type `git checkout first-step`

## A real world example
Next, let's create a script that actually does something useful for us.

We will create the before mentioned script that divides a text frame with multiple lines into separate text objects for each line. In your active Illustrator document, place at least one text frame for our script to operate upon.

Let's start by using the console to collect some necessary information about our text frames. Also, we only want to operate on elements that we selected in Illustrator. For that, we will use these lines of code to start with:
 ```javascript
// We want to operate on the currently active document
var doc = app.activeDocument;
// and use only the selected objects
var selection = doc.selection;

// loop through all selected objects
for (s = 0; s < selection.length; s++) {
  currentObject = selection[s];

  // make sure we operate only on text frames
  if (currentObject instanceof TextFrame) {
    // tell me how many lines of text the text frame has
    $.writeln(currentObject.lines.length, ' lines');
  }
}
```
_script reference:_ `git checkout count-lines`

Great! Now we already know one crucial information: it tells us how many lines a text frame includes. That number will also be the number of times that we create a new text frame on the artboard, each one only with the content of one line.


```javascript
var doc = app.activeDocument;
var selection = doc.selection;

for (s = 0; s < selection.length; s++) {
  currentObject = selection[s];

  if (currentObject instanceof TextFrame) {
    // get all lines included in the text frame
    var lines = currentObject.lines;

    // loop through all the lines
    for (l = 0; l < lines.length; l++) {
      // create a new empty text frame
      var newText = doc.textFrames.add();
      // duplicate the content of the current line to the new text frame
      lines[l].duplicate(newText.textRange, ElementPlacement.PLACEATBEGINNING);
    }
  }
}
```
_script reference:_ `git checkout divide-text-1`

Well, that piece of code already does what it is supposed to: it creates a new text frame for each line of the original selected text frame. Although not quite how we want it to be, because it places them all on top of each other in the upper-left corner of our artboard.  
But we want them placed in their original position! For that we have to know the original position of each line.

Our challenge is that the `Line` object does not have a position information, only the parent `TextFrame` has. Luckily our text frame also provides information about its line-height. Or rather the `CharacterAttributes` object living inside the text frame's `TextRange` object. We can reach it like this: 

```javascript
var lineHeight = currentObject.textRange.characterAttributes.leading;
```

Putting it all together, our script will look like this:
```javascript
var doc = app.activeDocument;
var selection = doc.selection;

for (i = 0; i < selection.length; i++) {
  currentObject = selection[i];

  if (currentObject instanceof TextFrame) {
    var lines = currentObject.lines;

    // get line height
    var lineHeight = currentObject.textRange.characterAttributes.leading;

    for (l = 0; l < lines.length; l++) {
      var newText = doc.textFrames.add();

      // set position of new text frame
      newText.left = currentObject.left;
      newText.top = currentObject.top - (l * lineHeight);

      lines[l].duplicate(newText.textRange, ElementPlacement.PLACEATBEGINNING);
    }

    // after we've copied all the lines, remove the original text frame
    currentObject.remove();
  }
}
```
_script reference:_ `git checkout divide-text-2`

That's it! We now have a working script that spares us the effort of cutting and pasting every single line of a list that we copied into Illustrator. With our handmade scripts, there's no more dull and repetitive clickwork to be done. Just foralize what you would do by hand into some lines of code and you will save yourself a big deal of time.  

You can find the complete JavaScript reference and other useful resources in the [scription section][url-reference] at Adobe. Play around with these methods and the next time you find yourself doing something again and again by hand, have a look into the reference and see if there's a method that you can use to automate that workload.


[url-reference]: http://www.adobe.com/devnet/illustrator/scripting.html