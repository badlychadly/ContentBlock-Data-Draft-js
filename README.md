# ContentBlock-Data-Draft-js

Draft-js can be a little tricky sometimes, and it can be difficult sometimes to add the specific functionality you are looking for. In a previous post I wrote about adding an entity of title to send the data to the server, but this seems a little unconventional for the task at hand. A better option may be just to save that text in the data property of the contentBlock. The challenging part of this is adding to that data property everytime I type into the title block. This post will go over what I have so far.

## HandleBeforeInput
I need a way of checking if the current selection is the first block in the contentState. Cancelable Handlers from Draft-js seem like a good way to hook into the event and see if selection is in the intended block. I did not want crowd my handleKeyCommand function so I went with the `handleBeforeInput` handler prop. This takes an argument of `char` is the character that was inputed, the editorState and an eventTimeStamp. To use it you declare a function with the intended logic, return a string of `handled` (which will prevent the default draft-js behavior ) or a string of `not-handled` (to tell Draft to do it's normal behavior), then pass it as a prop to the Editor Component.
```
handleBeforeInput = (chars, editorState) => {
  //logic
  }
  
  //EditorComponent
  
  <Editor
    ref="editor"
    editorState={this.state.editorState}
    keyBindingFn={this.keyBindingFn}
    blockStyleFn={getBlockStyle}
    handleKeyCommand={this.handleKeyCommand}
    handleBeforeInput={this.handleBeforeInput}
    onChange={this.onChange}
    plugins={this.plugins}
    placeholder="Hello"
  />
```

## Modifiers for Content
In order to make changes to the ContentState a Modifier must be used or you could run into potential problems. The specific Modifier method I intend to use is `mergeBlockData` which will help manage the data property of my contentBlock. This seemed perfect and easy enough but I ran into a snag. Since I am using `handleBeforeInput` which returns 'handled' or 'not-handled' Draft will either prevent it's normal process of adding characters and styles, or will override your added state making the changes that were made pointless. When 'handled' is returned the process of actually adding characters on the draft side is stopped, so I am unable to add any more characters to that block. To help deal with this I found another Modifier method called `insertText` which will insert the text I need to be added. This method takes a ContentState, TargetRange(Selection), and the string of text you want to be inserted along with optional parameters of inlineStyle and entityKey. Together it looks something like this:
```
handleBeforeInput = (chars, editorState) => {
    const selectionKey = this.state.editorState.getSelection().getStartKey()
    const contentState = this.state.editorState.getCurrentContent()
    const firstBlock = contentState.getFirstBlock()
    
        // See if we are inside the correct block //
    if(selectionKey === firstBlock.getKey()) {
        if (firstBlock.getLength() > 2) {
            const selection = this.state.editorState.getSelection()
            
              // INSERT THE TEXT //
            const addText = Modifier.insertText(contentState, selection, chars)
            
              // Merge The Data //
            const conStateBlockData = Modifier.mergeBlockData(addText, selection, {title: addText.getFirstBlock().getText()})
            
              // PUSH CHANGES TO THE EDITORSTATE //
            const newState = EditorState.push(this.state.editorState, conStateBlockData, "change-block-data")
            
              // MOVE SELECTION TO THE END OF THE BLOCK //
            const adjustSelection = EditorState.moveFocusToEnd(newState)
            debugger;
            console.dir(newState.getLastChangeType)
           this.onChange(adjustSelection) 
           return "handled"

        }
    }
  }
```
## Final Thoughts
The first step was getting this to work but now I need to fix some major bugs. The first being the prevention of styles I wish to add to the block. Another bug is if I want to make a change to the title block after I type a character the selection is forced to the last block of the Editor. I will try making use of targetRange parameter for `insertText` that way I don't use `moveFocusToEnd`. I will keep working of perfecting this functionality and improving my Draft-js skills.
