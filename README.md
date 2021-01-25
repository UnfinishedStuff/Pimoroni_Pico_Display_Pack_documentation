# Pimoroni_Pico_Display_Pack_documentation

![A Pico Display showing "Pimoroni display pack!"](/Display_pack.jpg)


Pimoroni have released a whole raft of accessories for the Raspberry Pi Pico, with the RP2040 chip at its heart.  Unfortunately, they seem to have released so many accessories that the documentation is currently a bit behind.  This document contains a description of what I've been able to figure out for using the MicroPython module provided [by Pimoroni here](https://github.com/pimoroni/pimoroni-pico) for their Display Pack.

Current state:  The first draft is done, and some feedback has been recieved (thanks Gadgetoid and GlennHoran!).  Currently tested on pimoroni-pico v0.0.5 Alpha, all functions present and working.  Redrafting done down to the buttons (inclusive).

# Quickstart script

<details>
    <summary>If you're reasonably comfortable with MicroPython then this example script lays everything out in brief.  It can be used directly on the Pico, and runs through every function in the module.</summary>
    
```python
# Import the module for the Display board
import picodisplay as display

# INITIAL SETUP
# Initialise the Display board.  This must be done for any features to work.
# Get the width of the display, in pixels (240 pixels)
width = display.get_width()
# Get the height of the display, in pixels (135 pixels)
height = display.get_height()

# Use the above to create a buffer for the screen, 2 bytes per pixel
# (it's a 16bit/RGB565 display, not RGB888)
display_buffer = bytearray(width * height * 2)
# Start the board!
display.init(display_buffer)

# SETTING THE LED
# Set the RGB LED.  r,g,b = ints of 0-255.  Effective immediately.
display.set_led(51, 153, 255) # A nice blue

# CHECKING THE BUTTONS
# Check button states. A,B,X,Y map to 0,1,2,3, or the constants BUTTON_A,
# BUTTON_B, BUTTON_X, BUTTON_Y.  Note: not interrupt driven.
if display.is_pressed(display.BUTTON_A):
    print("Button A is pressed!")

# USING THE SCREEN
# The screen backlight starts at 0.  Must be 0.0-1.0.
display.set_backlight(0.5)

# All drawing actions are done with a pen which defines a colour
display.set_pen(102, 255, 102) # A nice green

# Create pen variables to avoid having to remember what shade the numbers are
black = display.create_pen(0,0,0)
white = display.create_pen(255,255,255)
display.set_pen(black)

# Set the entire screen to the current pen colour
display.clear()

# No draw commands take effect instantly: you need to push data to the screen
display.update()

# Draw one pixel in current pen colour.  Params are X and Y coords in pixels
display.set_pen(white) # Remember to change your pen colours!
display.pixel(1,1)
display.update()

# Draw a horizontal line in current pen colour. Params are starting X and Y
# co-ordinates, and length of the line in pixels
display.pixel_span(0,3,240)
display.update()

# Draw a rectangle.  Params are top-left X and Y co-ords, width and height.
# Rectangle is filled with the current pen colour.
display.rectangle(0,5,25,25)
display.update()

# Draw a circle.  Params are centre X and Y co-ords and radius in pixels.
# The circle is filled with the current pen colour
display.circle(15,45,15)
display.update()

# Draw individual characters.  First param is the ASCII number for the char.
# Second and third are top/left X/Y coords.  Optional 4th param is font size,
# defaults to 2/~11px tall/~12 rows, 3 = ~20px/ 5rows, 4 = ~30px/ 4rows 
display.character(65, 0, 61)
display.character(66, 15, 61, 4) # With font size
display.update()

# Draw a string. First param is the string, two and three are upper left X/Y
# co-ords, four is wrapping: after each word is drawn, if the text is wider
# than w pixels wide then the next word is moved to the line below.  Optional
# param five is font size, see character function description.
display.text("Hello world!", 0,100, 240,4)
display.update()

# Clips define a rectangle inside which things can be drawn.  Anything drawn
# outside this area while the clip is present is not rendered.  Can be used to
# e.g. slice objects in half.  First two params are X/Y co-ords of upper left,
# next two are width and height.
display.set_clip(125,15,50,50)
display.circle(125,35,15) #Circle is on left hand clip border, so half removed
display.remove_clip()
display.update()
```
</details>

# Setting up the board and development tools

Before you can do anything, if you haven't already done so, you need to install the MicroPython firmware which contains Pimoroni's modules.  Follow [the instructions they have here](https://github.com/pimoroni/pimoroni-pico/blob/main/setting-up-micropython.md) to do this.  

I've been using [Thonny IDE](https://thonny.org/) to work with the board, which is the IDE recommended by the Raspberry Pi Foundation.  Plug the board into your computer with a micro-USB cable, and after installing Pimoroni's firmware as above, open Thonny.  In the bottom right of the IDE it will probably say `Python` followed by a version number (e.g. `3.7.9`).  Click on that, and select `MicroPython (Raspberry Pi Pico)`.  Type the code you want to run into the main window of Thonny, and when complete click on the Save icon just below and right of the Edit option (or use the keyboard shortcut, Ctrl-S).  This should ask you whether you want to save the code on `This computer` or the `Raspberry Pi Pico`, so select the Pico. If it asks you for a file name, I'd suggest calling it `main.py`: this special file name means the script should run any time the board is powered on.  You can also start and stop the script using the green Run and red Stop button at the top of Thonny.

# Setting up the Display Pack

Any time you use the Display Pack you'll first need to use a bit of code to get the board ready for use.  While the board is called the Display pack it also has four buttons and an RGB LED, and you'll need to go through these initialisation steps even to use the LED or buttons.  This takes a few steps, and the whole chunk of code which is required appears in one block at the bottom of this section

### Importing the picodisplay module

First, you need to tell MicroPython to use the package of software tools which Pimoroni have created for the Display Pack, using `import picodisplay as display`.  You _could_ just use `import picodisplay`, but then every time you wanted to use a tool from this package you'd have to start the command with `picodisplay`.  By importing it `as display` you can start the command with `display` instead, which is a bit shorter, and that's what we'll be doing throughout the rest of this documentation.  If you really want to save yourself a few keystrokes, you could do something like `import picodisplay as pds` and then prefix every command with `pds`.

### Creating a block of memory to act as a canvas to draw on: `display.get_width()` and `display.get_height()`

Next, we need to create a block of memory to hold the image the display will show.  This needs to be big enough to hold the whole image, so we need to know how big that is.  You could just look up the number of pixels in the screen on the product page and try to remember it, but the display can helpfully tell you how big it is. Using `width = display.get_width()` fetches the width of the screen in pixels and stores it in a variable called width, and `height = display.get_height()` predictably does the same for the height.  We need two bytes of memory for each of the pixels, so we create a `bytearray` to store the data (which is `width * height * 2` in size) and call that `display_buffer`.  A `bytearray` is simply a more basic structure for storing data in than a Python list, which makes it more useful for certain types of hardware.

```python
width = display.get_width()
height = display.get_height()
display_buffer = bytearray(width * height * 2)
```
### Starting the board with `display.init()`

Finally, we finish our setup by initialising the board with the buffer for the pixels using `display.init(display_buffer)`.  That's all the setup done!  This whole block of setup code is below, and should be at the top of any program you write which uses the Display Pack.  Now we can get to the fun stuff.

```python
# Boilerplate code which will be needed for any program using the Pimoroni Display Pack

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

# Using the Display Pack: the RGB LED
### Creating colour with `display.set_led(red, green, blue)`

The RGB LED on the board is really straightforward to use.  Like most controllable RGB LEDs, it takes three values, each between 0 and 255.  The first controls the amount of red light, the second the green, and the last the blue, and you can blend these together to create a vast range of colours.  The command for this is `display.set_led(red, green, blue)`, and the colour of the LED should change as soon as this command is run.  On an electrical level the LED is an analog LED with three different elements.  Each element is a different colour, and the strength of the colour is controlled by Pulse Width Modulation (PWM), a system of toggling a pin on and off to signal an intensity.

<details>
    <summary>Click here for a full example of using the LED.  This code can be used directly in Thonny.</summary>
    
```python    
# A complete example for using the LED

# Import a module which will let us wait for a few seconds
import time

#Standard boilerplate code for using the Display Pack
import picodisplay as display
width = display.get_width()
height = display.get_height()
display_buffer = bytearray(width * height * 2)
display.init(display_buffer)

# And now, lets cycle the LED through a few colours
display.set_led(255,0,0) # Set the LED to bright red
time.sleep(1) # Wait for 1 second

display.set_led(0,255,0) # Set the LED to bright green
time.sleep(1) # Wait for 1 second

display.set_led(0,0,255) # Set the LED to bright blue
time.sleep(1) # Wait for 1 second

display.set_led(0,0,0) # Turn the LED off
```
</details>

# Using the Display Pack: the buttons
### Checking to see if a button is pressed with `display.is_pressed()`

At the moment the only way to control the buttons seems to be to check if they're pressed individually.  If you're familiar with interrupts it seems you'll need to look into the base MicroPython documentation to do that, as they aren't catered for here.

To check if a button is pressed, use the `display.is_pressed(button)` function;  `button` identifies which button to check.  The buttons on the board are labelled `A,B,X,Y` and you can use `display.BUTTON_A`, `display.BUTTON_B`, `display.BUTTON_X` and `display.BUTTON_Y` to tell the board which one to check (you _can_ refer to them as 0,1,2, and 3 for A,B,X and Y, but the pre-defined `BUTTON_` names are much more easy to remember and read back).  This will then return `True` or `False` depending on whether or not the button is pressed.

```python
while True:
    if display.is_pressed(display.BUTTON_A):
        print("Button A is pressed!")
    else:
        print("Button A isn't pressed...")
    time.sleep(0.5)
```

Bear in mind that this example is _not_ interrupt-driven, which means that if the button isn't pressed at the _exact_ time the code checks for it, nothing will happen and the code may miss the button press.  You'll need to constantly check to see if the button is pressed.  Once I've had more time to tinker with this I'll see if I can put together an interrupt-driven version which should catch a button press as soon as it happens.

<details>
    <summary>Click here for a full example of using the buttons.  This code can be used directly in Thonny.</summary>

```python
# A complete example for using the buttons

# Import a module which will let us wait for a few seconds
import time

#Standard boilerplate code for using the Display Pack
import picodisplay as display
width = display.get_width()
height = display.get_height()
display_buffer = bytearray(width * height * 2)
display.init(display_buffer)

# And now, lets check each button one-by-one for presses, and print a message if it is pressed
    
while True:                                     # Continuously check for button presses
    if display.is_pressed(display.BUTTON_A):    # Check if the A button is pressed
        print("Button A is pressed!")               # If button A is pressed, print a message saying so!
    elif display.is_pressed(display.BUTTON_B):  # Otherwise, check if the B button is pressed
        print("Button B is pressed!")               # If button B is pressed, print a message saying so!
    elif display.is_pressed(display.BUTTON_X):  # Otherwise, check if the X button is pressed
        print("Button X is pressed!")               # If button X is pressed, print a message saying so!
    elif display.is_pressed(display.BUTTON_Y):  # Otherwise, check if the Y button is pressed
        print("Button Y is pressed!")               # If button Y is pressed, print a message saying so!
    else:                                       # If none of the buttons are pressed..
        print("No buttons are pressed...")          # Print a message saying that no buttons are pressed
    time.sleep(0.2)                             # Wait a moment so that the messages don't scroll by too quickly.
```
</details>

# Using the Display Pack: the display!

OK, so now to the part you probably bought the board for.  This part is going to be a bit more complex, but bear with me.  We'll need to get a few sections in before we'e covered enough code to form a complete example.

## Any colour of pen you want: `display.set_pen(r,g,b)` and `display.create_pen(r,g,b)`

The way to think about this is that you're drawing on the screen with a pen.  Before you can do this, you need to pick which colour of pen you want to use.  This is what the `display.set_pen(r,g,b)` command is for.  `r`,`g`, and `b` should each be values between 0 and 255, and like the LED the values will control the overall colour of the pen used to draw on the screen.  Interestingly, if you _don't_ set a pen colour it seems to automatically give you a random colour for every draw action you take.

You could use this to draw everything you want to do on the screen, changing colour as required.  If you're using a lot of different colours though it could get confusing to remember exactly _which_ shade of colour each combination of `r`, `g`, and `b` values refer to, and so there's a shortcut for this.  The `display.create_pen(r,g,b)` function is used for this, for example by using the line of code: `penName = display.create_pen(r,g,b)`. In this example `penName` is the shortcut name for this colour, so the next time you need it you can use the command `display.set_pen(penName)` instead of trying to remember the exact `r`, `g`, and `b` values.

<details>
    <summary>This example shows how to set pen colours and create shortcut pens, but won't display anything on the screen by itself.  You'll need to combine it with some of the code in the next sections to start seeing things.</summary>
    
    # An example of setting pen colours and creating pens

    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # And now we'll start setting pen colours
    display.set_pen(255,0,0)    # Set the pen to bright red
    display.set_pen(0,255,0)    # Set the pen to bright green
    display.set_pen(0,0,255)    # Set the pen to bright blue
    
    brightBlue = display.create_pen(0,0,255)    # Create a shortcut for bright blue
    darkBlue   = display.create_pen(0,0,128)    # Create a shortcut for dark blue
    
    display.set_pen(brightBlue)                 # Quickly set the pen to bright blue
    display.set_pen(darkBlue)                   # Quickly set the pen to dark blue
    
    
</details>

## Setting whole-screen brightness: `display.set_backlight(brightness)`

Sometimes you might want the whole screen to go through changes in intensity, and trying to do that at the pixel level could be difficult.  Alternatively, sometimes you just want the whole screen to be a bit dimmer, perhaps in a dark room.  The `display.set_backlight(brightness)` will do this, where `brightness` should be a value from 0 (backlight off completely) to 1 (backlight on full).  This change will take effect immediately.

**Important note**: when the display is initialised the backlight seems to be turned off.  You'll likely need to set the backlight to a higher brightness before anything will be seen on the screen.  If you're pushing data but not seeing anything, check your backlight value.

There's no example for setting the backlight value because you'll see absolutely nothing just by doing that, so see the next example.


## Draw individual pixels: `display.pixel(x,y)` and `display.update()`

Once you've set your pen colour you can begin drawing pixels.  The most simple way to do this is to set the colour of individual pixels.  This is done with the `display.pixel(x,y)` command.  The `x` and `y` values are coordinates, and describe where on the screen the pixel should be drawn, where the X axis is the long edge of the screen and the Y axis is the short edge.  The pixel will be drawn in the colour of the current pen.

However, when you use the pen to draw, nothing will immediately happen on the screen.  That's because you're really drawing to the buffer we created earlier, but not sending that buffer to the screen.  To send the data in the buffer to the display you need to use the `display.update()` function.  This allows you to write more complex programs with many separate drawing instructions, without each individual step showing up on the screen: the final result will show on the screen only when you're ready for it to appear.

<details>
    <summary>This example shows how to set individual pixels and then push these to the screen, and is the first example where you should be able to see the results on the screen.</summary>
    
    # An example of setting individual pixels and pushing them to the screen

    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # And now we'll start setting pixels!
    display.pixel(0,0)  # Draw one pixel at coordinate 0,0, the top left of the screen
    display.pixel(5,5)  # Draw one pixel 5 pixels across, and 5 down
    
    # Push the dranw pixels to the screen:
    display.update()
    # You should now be able to see two tiny red dots on the display!
    
</details>

## Filling the screen with a colour: `display.clear()`

When you've spent plenty of time writing to the screen you may want to start over with a clean slate.  You can achieve this using the `display.clear()` function.  As the name implies, this can be used to clear the whole screen, but it actually sets the whole screen to the colour of the current pen.  That means it can also be used to set colourful backgrounds to draw other shapes onto as well.  Just bear in mind that, like other drawing actions, this won't appear on the screen until you use `display.update()`.

<details>
    <summary>This example shows how you can set the entire screen to the colour of a pen</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Set the whole screen to the current pen colour
    display.clear()
    
    # Push the pixels to the screen:
    display.update()
    # The entire screen should now be red.
    
</details>

## Drawing more pixels in one go: `display.pixel_span(x,y,,)`

Drawing individual pixels will quickly get tedious.  You can draw straight lines using the `display.pixel_span(x,y,l)` function.  `x` and `y` are the starting coordinates of the line, and `l` is the length of the line to draw, in pixels, using the current pen colour.  Unfortunately this only seems to work for horizontal lines at the moment: vertical lines will need to be done with the `display.set_pixel()` function.

<details>
    <summary>See how you can draw straight lines using this example</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Draw a line starting 5 pixels down and 5 pixels from the edge, 100 pixels long
    display.pixel_span(5,5,100)
    
    # Push the pixels to the screen:
    display.update()
    # You should now see a long, straight line on the screen
    
</details>

## Drawing MORE pixels in one go: `display.rectangle(x,y,w,h)`

If you need to draw a rectangle (it works for squares too!) you can use the `display.rectangle(x,y,w,h)` function.  `x` and `y` are the coordinates of the top-left of the rectangle on the screen, `w` is the width of the screen in pixels, and `y` is the height.  The whole rectangle will be filled with the current pen colour.

<details>
    <summary>See how you can draw straight lines using this example</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Draw a rectangle.  The top left corner should be 5 pixels from the top and 5 from the edge.
    # The rectangle should be 100 pixels wide, and 100 tall.
    display.rectangle(5,5,150,100)
    
    # Push the pixels to the screen:
    display.update()
    # You should now see a red rectangle on the screen
    
</details>

## Drawing MOAR pixels, but in a circle: `display.circle(x,y,r)`

Circles can be a pain to do manually because you have to try to fit a round object into square pixels.  Thankfully, there is a function to figure out the best way to do this for you: `display.circle(x,y,r)`.  In this case `x` and `y` are the coordinates of the centre of the circle, and `r` is the radius in pixels.  The whole circle will be filled in the current pen colour.

<details>
    <summary>Let's draw a circle in this example!</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Draw a circle.  The centre of the circle should be 50 pixels down and 50 pixels from the edge.
    # The radius of the circle will be 25 pixels
    display.circle(50,50,25)
    
    # Push the pixels to the screen:
    display.update()
    # You should now see a red circle on the screen
    
</details>


## Drawing EVEN MOAR pixels, but as letters: `display.character(c, x, y)`

At some point you'll probably want to write some text on the screen, whether it be a sensor reading, some information, or just plain old `Hello World!`.  To print individual characters on the screen you can use the `display.character(c,x,y)` function.  Here, `c` is the number used to refer to the character in an ASCII table.  If you've not heard of the ASCII table, it's a standard for encoding characters on a computer where every character has an ID number.  Use your favourite search engine to look up "ASCII tables", and you'll find a plethora of website which will show you the table which you can use to look up the ID of the character you want to print.  Upper and lower case letters are encoded separately, so for example the letter `A` is number 65, whereas the letter `a` is 97.  The `x` and `y` parameters in the function are the coordinates of the upper left of the character, and the letter will be written in the current pen colour.

There's also an optional fourth parameter for this function, which is the _scale_ essentially the font size.  The default seems to be 2, which gives characters about 11 pixels tall, so at an absolute maximum you'll get 12 cramped rows of text.  A scale of 3 gives characters about 20 pixels tall/5 rows, and 4 is just under 30-pixel tall characters/ about 4 rows of text. 

<details>
    <summary>This example shows how to print individual characters on the screen.</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Draw the character "A", 5 pixels from the top and 5 from the edge
    display.character(65, 5,5)
    # Do the same as above, but 15 pixels in and down, and in fontsize 4
    display.character(65, 15,15,4)
    
    # Push the pixels to the screen:
    display.update()
    # You should now see two red "A"s on the screen. 
    
</details>

## Drawing strings of text: `display.text(string,x,y,w)`

Drawing individual characters for words and sentences would rapidly get tedious, so the `display.text(stringx,y,w)` function will draw whole sentences for you.  `string` should be the string of text you want to display (e.g. `"Hello world!`), while `x` and `y` specify the upper left-hand corner of the text box.  `w` is the wrap value.  If you have multiple words separated by spaces, then the function will check the width of the text on the screen after each word is written.  If the width is greater than `w` pixels, then the next words will automatically be moved to the line below.  Note that this will not split up individual words, it will only move subsequent words to the next line.

This function will also take an optional font size, see the "Drawing EVEN MOAR pixels, but as letters: `display.character(c, x, y)`" section above for an explanation.

<details>
    <summary>This example shows the classic "Hello world!" example on the display with different wrapping</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Draw "Hello world!" on the screen 5 pixels in and down from the top left corner.
    #The Wrap width of 200 is wider than the text, so the text won't wrap.
    display.text("Hello world!", 5, 5, 200)
    
    
    # Draw "Hello world!" on the screen 5 pixels in and 50 down from the top left corner.
    #The Wrap width of 100 is narrower than the text, so the text will automatically
    # wrap onto two lines
    display.text("Hello world!", 5, 50, 100)
    
    # Push the pixels to the screen:
    display.update()
    # You should now see "Hello world!" on the screen with different wrapping. 
    
</details>


<details>
    <summary>This example shows the classic "Hello world!" example in different font sizes</summary>
    
    #Standard boilerplate code for using the Display Pack
    import picodisplay as display
    width = display.get_width()
    height = display.get_height()
    display_buffer = bytearray(width * height * 2)
    display.init(display_buffer)

    # Set the backlight to 50%
    display.set_backlight(0.5)

    # Use a red pen:
    display.set_pen(255,0,0)

    # Draw "Hello world!" on the screen 5 pixels in and down from the top left corner.
    #The Wrap width of 240 is wider than the text, so the text won't wrap.  The font
    # size is 1, which is tiny!
    display.text("Hello world!", 5, 5, 240,1)
    
    
    # Draw "Hello world!" on the screen 5 pixels in and 50 down from the top left corner.
    #The Wrap width of 100 is narrower than the text, so the text won't wrap.  The font
    # size is 4, which is much bigger
    display.text("Hello world!", 5, 50, 240,4)
    
    # Push the pixels to the screen:
    display.update()
    # You should now see "Hello world!" on the screen in two different sizes. 
    
</details>

## Advanced drawing: `display.set_clip(x,y,h,w)` and `display.remove_clip()`

The functions above lay out some fairly basic drawing tools: setting individual pixels, drawing filled rectangles, and drawing filled circles.  These can be used together to create more complex shapes, but they're "additive": you can draw a whole circle, but you can't draw half a circle.  To help create more complex objects you can use a _clip_.  This defines a part of the screen which can be drawn on, and anything outside this area cannot be drawn on.  Think of this as "masking off" parts of the screen with masking tape so that drawing on the masked area won't show up once the mask is removed.  The `display.set_clip(x,y,h,w)` function will create a rectangular clip with `x` and `y` specifying the upper left corner of the clip, and `w` and `h` defining the width and height of it.  Only drawing actions within this area will eventually appear on the screen: nothing will appear outside it.  When you're finised, use `display.remove_clip()` to remove the clip and enable drawing to any part of the screen again.

This diagram will hopefully explain what clips do a bit better:
![A diagram explaining clips](/clip.jpg)

