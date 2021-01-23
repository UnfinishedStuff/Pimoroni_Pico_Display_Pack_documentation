# Pimoroni_Pico_Display_Pack_documentation


Pimoroni have released a whole raft of accessories for the Raspberry Pi Pico, with the RP2040 chip at its heart.  Unfortunately, they seem to have released so many accessories that the documentation is currently a little scarce.  This doc contains a description of what I've been able to figure out for how to use the MicroPython module provided [by Pimoroni here](https://github.com/pimoroni/pimoroni-pico) for their Display Pack.

Current state:  the functions are all documented, but it's getting late so complete examples will have to wait until later.

# Setting up the board and development tools

Before you can do anything, if you haven't already done so, you need to install the MicroPython firmware which contains Pimoroni's modules.  Follow [the instructions they have here](https://github.com/pimoroni/pimoroni-pico/blob/main/setting-up-micropython.md).  I've been using [Thonny IDE](https://thonny.org/) to work with the board, which is the IDE recommended by the Raspberry Pi Foundation.  Plug in the board, and after installing Pimoroni's firmware, open Thonny.  In the bottom right of the IDE it will probably say `Python` followed by a version number.  Click on that, and select `MicroPython (Raspberry Pi Pico)`.  Type code you want to run in the main window, and when complete click on the Save (floppy disk, or Ctrl-S on your keyboard) icon just below and right of the Edit option.  This should ask you whether you want to save the code on your PC or the Pico, so select the Pico.  The code won't run immediately:  you'll have to click the green run button (or press F5 on your keyboard) to start the program.

# Setting up the display

Any time you use the Display Pack you'll need to use a bit of code to get the board ready for use.  

First, you need to tell MicroPython to use the package of software which Pimoroni have created for the Display pack, using `import picodisplay as display`.  You _could_ just use `import picodisplay` but then every time you wanted to use a tool from this package you'd have to start the command with `picodisplay`.  By importing it `as display` you can start the command with `display` instead, which is a bit shorter.  If you really want to save yourself a few keystrokes, you could do something like `import picodisplay as pds`.

Next, we'll need to create a block of memory to hold the image the display will show.  This needs to be big enough to hold the whole image, so we need to know how big that is.  You could just look the number of pixels in the screen up on the product page and try to remember it, but the display can helpfully tell you how big it is. `width = display.get_width()` fetches the width of the screen in pixels and stores it in a variable called width, and `height = display.get_height()` predictably does the same for the height.  We need two bytes of memory for each of the pixels, so we create a `bytearray` of pixels `width * height * 2` in size and call that `display_buffer`:

```
width = display.get_width()
height = display.get_height()
display_buffer = bytearray(width * height * 2)
```

Finally, we finish our setup by initialising the board with the buffer for the pixels using `display.init(display_buffer)`.  That's all the setup done!  This whole block of setup code is below, and after that we'll get to the fun stuff.

```
# Import the module containing the display code
import picodisplay as display

# Get the width of the display, in pixels
width = display.get_width()
# Get the height of the display, in pixels
height = display.get_height()

# Use the above to create a buffer for the screen.  It needs to be 2 bytes for every pixel.
display_buffer = bytearray(width * height * 2)

# Start the display!
display.init(display_buffer)
```

# Using the display: the RGB LED

The RGB LED on the board is really straightforward to use.  Like most controllable RGB LEDs, it takes three values, each between 0 and 255.  The first controls the amount of red light, the second the green, and the last the blue.  The command for this is `display.set_led(red, green, blue)`, and the colour of the LED should change as soon as this command is run.  On a technical level the LED isn't a Neopixel, it is an analog LED with three different elements.  Each element is a different colour, and the strength of the colour is controlled by PWM.

# Using the display: the buttons

At the moment the only way to control the buttons seems to be to check if they're pressed individually.  If you're familiar with interrupts it seems you'll need to look into the base MicroPython documentation to do that, as they aren't catered for here.

To check if a button is pressed, use the `display.is_pressed(button)` function.  `button` should be a number.  The buttons on the board are labelled `A,B,X,Y` but in the software they're referred to as `1,2,3,4`, and you should replace `button` with a number.  This will then return `True` or `False` depending on whether or not the button is pressed.

```
while True:
    if display.is_pressed(0):
        print("Button A is pressed!")
    else:
        print("Button A isn't pressed...")
    time.sleep(0.5)
```

Bear in mind that this example is _not_ interrupt-driven, which means that if the button isn't pressed at the _exact_ time the code checks for it, nothing will happen and the code may miss the button press.  You'll need to constantly check to see if the button is pressed.  Once I've had more time to tinker with this I'll see if I can put together an interrupt-driven version.

# Using the display: the display!

OK, so now to the part you probably bought the board for.  This part is going to be a bit more complex, but bear with me.

## Any colour of pen you want

The way to think about this is that you're drawing on the screen with a pen.  Before you can do this, you need to pick which colour of pen you want to use.  This is what the `display.set_pen(r,g,b)` command is for.  `r`,`g`, and `b` should each be values between 0 and 255, and like the LED the exact values will control the overall colour of the pen used to draw on the screen.

You could use this to draw everything you want to do on the screen, changing colour as required.  If you're using a lot of different colours though it could get confusing to remember exactly _which_ shade of colour each combination of `r`, `g`, and `b` values refer to, and so there's a shortcut for this.  The `display.create_pen(r,g,b)` function is used for this, for example by using the line of code: `penName = display.create_pen(r,g,b)`. In this example `penName` is the shortcut name for this colour, so the next time you need it you can use the command `display.set_pen(penName)` instead of trying to remember the exact `r`, `g`, and `b` values.

## Draw individual pixels: `display.set_pixel(x,y)`

Once you've set your pen colour you can begin drawing pixels.  The most simple way to do this is to set the colour of individual pixels.  This is done with the `display.set_pixel(x,y)` command.  The `x` and `y` values should be numbers and describe the pixel which should be set the colour of the pen, where the X axis is the long edge of the screen and the Y axis is the short edge.

## Pushing the data to the screen: `display.update()`

When you use the pen to draw, nothing will immediately happen on the screen.  That's because you're really drawing to the buffer we created earlier.  To send the data in the buffer you need to use the `dusplay.update()` function.  This allows you to write more complex programs with many separate drawing instructions, without each individual step showing up on the screen: the final result will show on the screen when you're ready for it to appear.

## Filling the screen with a colour: `display.clear()`

When you've spent plenty of time writing to the screen you may want to start over with a clean slate.  You can achieve this using the `display.clear()` function.  As the name implies, this can be used to clear the whole screen, but it actually sets the whole screen to the colour of the current pen.  That means it can also be used to set colourful backgrounds to draw other shapes on too.  Just bear in mind that, like other drawing actions, this won't appear on the screen until you use `display.update()`.

## Drawing more pixels in one go: `display.pixel_span(x,y,,)`

Drawing individual pixels will quickly get tedious.  You can draw straight lines using the `display.pixel_span(x,y,l)` function.  `x` and `y` are the starting coordinates of the line, and `l` is the length of the line to draw, in pixels, using the current pen colour.  Unfortunately this only seems to work for horizontal lines at the moment: vertical lines will need to be done with the `display.set_pixel()` function.

## Drawing MORE pixels in one go: `display.rectangle(x,y,w,h)`

If you need to draw a rectangle (it works for squares too!) you can use the `display.rectangle(x,y,w,h)` function.  `x` and `y` are the coordinates of the top-left of the rectangle on the screen, `w` is the width of the screen in pixels, and `y` is the height.  The whole rectangle will be filled with the current pen colour.

## Drawing MOAR pixels, but in a circle: `display.circle(x,y,r)`

Circles can be a pain to do manually because you have to try to fit a round object into square pixels.  Thankfully, there is a function to figure out the best way to do this for you: `display.circle(x,y,r`.  In this case `x` and `y` are the coordinates of the centre of the circle, and `r` is the radius in pixels.  The whole circle will be filled in the current pen colour.

## Drawing EVEN MOAR pixels, but as letters: `display.character(c, x, y)`

At some point you'll probably want to write some text on the screen, whether it be a sensor reading, some information, or just plain old `Hello World!`.  To print individual characters on the screen you can use the `display.character(c,x,y)` function.  Here, `c` is the number used to refer to the character in an ASCII table.  If you've not heard of the ASCII table, it's a standard for encoding characters on a computer where every character has an ID number.  Use your favourite search engine to look up "ASCII tables", and you'll find a plethora of website which will show you the table which you can use to look up the ID of the character you want to print.  Upper and lower case letters are encoded separately, so for example the letter `A` is number 65, whereas the letter `a` is 97.  The `x` and `y` parameters in the function are the coordinates of the upper left of the character, and the letter will be written in the current pen colour.

There's also an optional fourth parameter for this function, which is the _scale_ essentially the font size.  The default seems to be 2, which gives characters about 11 pixels tall, so at an absolute maximum you'll get 12 cramped rows of text.  A scale of 3 gives characters about 20 pixels tall/5 rows, and 4 is just under 30-pixel tall characters/ about 4 rows of text. 

## Drawing strings of text: `display.text(string,x,y,w)`

Drawing individual characters for words and sentences would rapidly get tedious, so the `display.text(stringx,y,w)` function will draw whole sentences for you.  `string` should be the string of text you want to display (e.g. `"Hello world!`), while `x` and `y` specify the upper left-hand corner of the text box.  `w` is the wrap value.  If you have multiple words separated by spaces, then the function will check the width of the text on the screen after each word is written.  If the width is greater than `w` pixels, then the next words will automatically be moved to the line below.  Note that this will not split up individual words, it will only move subsequent words to the next line.

## Advanced drawing: `display.set_clip(x,y,h,w)` and `display.remove_clip()`

The functions above lay out some fairly basic drawing tools: setting individual pixels, drawing filled rectangles, and drawing filled circles.  These can be used together to create more complex shapes, but they're "additive": you can draw a whole circle, but you can't draw half a circle.  To help create more complex objects you an use a _clip_.  This defines a part of the screen which can be drawn on, and anything outside this area cannot be drawn on.  Think of this as "masking off" parts of the screen with masking tape so that drawing on that area won't show up once the mask is removed.  The `display.set_clip(x,y,h,w)` function will create a rectangular clip with `x` and `y` specifying the upper left corner of the clip, and `w` and `h` defining the width and height of it.  Only drawing actions within this area will eventually appear on the screen: nothing will appear outside it.  When you're finised, use `display.remove_clip()` to remove the clip and enable drawing to any part of the screen again.

This diagram will hopefully explain what clips do a bit better:
![A diagram explaining clips](/clip.jpg)

## Whole-screen brightness: `display.backlight(brightness)`

Sometimes you might want the whole screen to go through changes in intensity, and trying to do that at the pixel level could be difficult.  Alternatively, sometimes you just want the whole screen to be a bit dimmer, perhaps in a dark room.  The `display.backlight(brightness)` will do this, where `brightness` should be a value from 0 (backlight off completely) to 1 (backlight on full).  This change will take effect immediately: there's no need to use the `display.update()` function.
