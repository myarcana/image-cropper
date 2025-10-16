# Design Considerations

- All 100% clientside, privacy-first; the local image you work on only stays in memory in the browser, the page never accesses the internet
- Mobile-first, these github-based widgets I'm building are all things I wish iOS provided out of the box tbh. Now at least I can access them if I have internet.
- Undo should also undo movement of a handle
- Undoing a cut/crop should restore the shape of the selection (really thankful for this, sometimes you accidentally hit crop when you meant cut, but it's so easy to undo then tap the other button)
- How to get image from the website straight into iOS Camera Roll? Skip Files. If Share Tray available, use it. If not, have a manual <a> tag (anti-popup proof) wrapping the download image button with the blob URL of the current image
- Thought custom setter was a good way to keep synchronised. I might throttle it.
- Use `<img>` tag so that image interactions work (like right click and save)
- 



-------

# Original Prompt (third iteration)

Write a singlepage vanilla HTML / responsive CSS / JavaScript static webpage that allows the user to "upload" an image (it just stays in memory, never gets sent to any server), then allows the drag handles to crop that image or cut sections out of that image, potentially make many edits in a row, and finally download their created image as a new image.

Do NOT use a canvas, just use an <img> tag to display the image as the user works on it, so that standard image interactions (like right click > save to image) work out of the box.

This webpage should be mobile-first: it should look good on phones. Desktop appearance is less of a priority, but it should be responsive.

The selection area should cover the whole image when the image is first loaded.

The selection area should be displayed as a thin white border with handles on the middle of each edge.

When the top or bottom handle are not at their respective edge of the image, the right and left handles should be disabled (hidden). Similarly, when the right or left handles are not both at the right and left of the image, the top and bottom handles should be disabled.

Implement that by:

- mousedown on handle activates handle dragging mode, remembers handle starting position
- every mousemove in handle dragging mode moves the selected handle. every mousemove: if the selected handle is at its edge of the image, and the opposite handle is too, then enable all four handles. Otherwise, disable the adjacent two handles.
- mouseup in handle dragging mode: turn off handle dragging mode. If end position different to starting position, push starting position to undo stack. If the handle that was being dragged is at its edge of the image and the opposite/paired handle is too, then enable all 4 handles. Otherwise disable the adjacent 2 handles.

It should be easy to move the handles to the edge of the image. You can do that by tracking mousemoves in the whole window and snapping the handles to the image edges when the mouse moves outside of the image. Don't fall into the trap of only assuming the user is able to move handles when they are clicking inside of the image element.

This excluding logic (pairing opposite handles together) means that: the selection area will always be full width or full height. It must be impossible to select a part of the image that does not span the image entirely in one of the vertical or horizontal directions. So the selection area always selects a "strip" of the image, from one edge to the opposite edge.

There should be a fixed floating toolbar (sticky) with these 4 buttons in it and no others:
* Undo
* Cut
* Crop
* Share
The buttons should be icons, not text.

Clicking Undo: undoes the last cut, crop action, or handle move (puts the handle back to where it was when the user clicked it) Clicking Redo: redoes that cut or crop or handle move. 
Clicking cut: removes the selected strip of the image from the image, and sticks the two remaining parts directly next to each other. For example, clicking cut while selecting a horizontal strip, will generate an image that is the visual concatenation of the content of the original image that was above the strip directly above the part of the image that was below the strip. Moves the handles to the edges of the new image.
Clicking crop: Removes everything from the image except for the selected strip. Moves the handles to the edges of the new image.

The user should be able to make as many cuts and crops in a row as they want. The displayed image always reflects the accumulation of all of their changes made so far. 

Clicking share: opens the sharing tray (if it is available) for the created image, otherwise opens the Save File dialog to prompt the user to save the image they've made

Implement undo like this using an undo stack with these specific steps:
 - When a handle is clicked, remember the starting position of this handle.
 - When a handle is released, if the release position of the handle is different to the starting position, then push "move handle": top/bottom/left/right handle, handle START location onto the undo stack (lifting a handle, dragging it around, then putting it back where it was all without letting go of the handle, should not create an undo stack item)
 - When a crop is performed, push "crop: pre-crop image, handle positions used for crop" onto the stack (then crop the image and put the handles at the edges of the new image)
 - When a cut is performed, push "cut: pre-cut image, handle positions used for cut" onto the stack (then cut the image and set the handles to the edges of the new image)
 - When undo is clicked, pop from the undo stack:
    - if it is move handle, then move the handle back to the location stored in this stack item.
    - If it is crop, then restore the image to the pre-crop image, and then put the handles to the positions used for crop.
    -  If it is cut, restore the image to the pre-cut image, and then put the handles to the positions used for cut. 

# Hand-implemented

I had to fix by hand:
- saving to iOS camera roll fallback if share tray doesn't work, clever `<a>` and `preventDefault` stuff and the setters in `state`
- the selection border (it originally used 4 oblong divs) and adding a black border around the white border so it's visible on white images too
- add `touch-action: manipulation` to buttons to disable zoom-spam when undoing a lot of stuff in a row
- disable scrolling and make sure the image will never overflow (not done yet)
- handle hitbox

It's pretty amazing how much the AI does by itself and how good it looks.
