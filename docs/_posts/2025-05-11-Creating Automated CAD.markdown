
This projects combines two of my biggest passions - 3D CAD designs and guitars, but unfortunately software as well.

I'll admit that If you share neither of those passions with me you don't have much to do in this article;
Keep reading on your own discretion.

## Automated CAD -
I’ve worked with SolidWorks, Fusion 360, and Onshape. They’re all powerful tools once you know how to use them, but they aren’t optimized for repetition. If you’re doing wire routing, mold design, or really anything that follows a pattern, manually updating sketches or features quickly becomes tedious.

All three products I mentioned has some programming interface to automate these types of tasks in a custom, user-defined manner.

 - Soildworks API uses VisualBasic, C# or C++ and is therefore deemed unusable for these purpose, as my patience is being tested enough already.

I chose Onshape over Fusion for this experiment for a very good reason - I tried it first and it worked well enough for me to give up trying the Fusion API.

Let's try to solve an actual problem!


# The Use case

While it's very much possible to design a guitar with pen and paper, many luthiers  choose CAD solutions in order to be able to use CNC machines in their manufacturing process.

The fretboard of the guitar (The top of the neck, the surface that holds the frets, where the player presses their fingers) requires special attention and involves some accurate dimensions to ensure player comfort and tonal accuracy.

![](https://cdn.mos.cms.futurecdn.net/rvdi3hkQaxuaK6bN5HyTQm.jpg)

Building a typical fretboard usually involves - 
 - Picking a hard piece of wood
 - Cutting it to shape - flat plank with wide base
 - Milling slots for the fret to fit into with a saw or a small milling bit
 - Curving the top of surface

Designing this in CAD is also mostly straightforward, except for the fret dimensions:
The distance between frets is very specific and is calculated using a formula that depends on the scale length (total length of the vibrating portion of the string) of the guitar.

![](https://us2.dh-cdn.net/uploads/db5587/original/3X/8/d/8d8d7b974e2927579844e3c8aca28fa598e53a77.jpeg)


Fretboards are a prime candidate for automation. Fret positions are mathematical and follow a consistent formula based on the scale length. Once you know how to compute them, it’s trivial to generate them for any scale. ’Whats not trivial is inputting them into CAD software by hand every time, even though most luthiers will end up with a **very** similar design. 

So I wrote a custom Onshape feature that generates a parametric fretboard using a few key inputs: scale length, fret count, radius, string count, and so on. This allows me to quickly create fretboards with varying specifications without having to redo any of the work.

[Source Code](https://github.com/LITzman/GenerativeGuitarFretboard) - Follow along as you read!

![](https://private-user-images.githubusercontent.com/31805301/358858194-2692ac04-5b85-4e1f-a043-29fb3fdf8932.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDY5OTI2ODYsIm5iZiI6MTc0Njk5MjM4NiwicGF0aCI6Ii8zMTgwNTMwMS8zNTg4NTgxOTQtMjY5MmFjMDQtNWI4NS00ZTFmLWEwNDMtMjlmYjNmZGY4OTMyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTExVDE5Mzk0NlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWIzNDFiMDllNDU1YTQxODAwMDgyMzhjZjNjY2JhMTY4ZmQ5YmU5NjBlZjU5OWVkMWMwZGViZWM0OWIyNzc3NzQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.pIRAq96u9WIRDau8z1R5YRA1EirtuySsMeQ6D0hW0Hc)


I even had the pleasure of [making a fretboard](https://imgur.com/a/0HkGCiF) that was designed by an experimental version of this code!

# FeatureScript Basics - 
Inside your Onshape project you can click "Insert new tab" on the bottom left corner of the page and than "Create Feature Studio".
You will now be greeted with a fresh IDE page where you will implement your feature.

Here are the basics - 
 - We are using a JavaScript-like syntax which is easy enough to figure out without actually knowing JavaScript.
 
 - The "Commit" button saves your progress and checks your code for errors.

 - You can test your code in a part studio by using "Add custom features" and browsing for your FeatureScript.

 - Documentation is here - https://cad.onshape.com/FsDoc/index.html

 - The debug-print function is `debug()` and it receive a context argument (which is given to your feature as an argument) and your custom input.

Click the "New feature" button and you'll receive something like this - 

```js
annotation { "Feature Type Name" : "My Feature", "Feature Type Description" : "" }
export const myFeature = defineFeature(function(context is Context, id is Id, definition is map)
    precondition
    {
        // Define the parameters of the feature type
    }
    {
        // Define the function's action
    });
```

Your feature is created by the defineFeature function and the `annotation` map suggest some strings that will be used for the UI of your feature.

 - The `context` argument represents a single part with all of it's bodies, geometries, errors, metadata etc...
During the creation of the feature we will enrich this context with new geomteries.

 - The `id` arguemnt is an identification string that is used internaly by Onshape for identification of sub-features.

 - The `definition` argument is a map of feature parameters that can be referenced throughout the creation of the feature.

My feature's defintion looks like this - 

```
annotation { "Feature Type Name" : "Generative Fretboard" }
export const generativeFretboard = defineFeature(function(context is Context, id is Id, definition is map)
```

# User parameters and UI
In the precondition scope of the feature we created will fill the definition map with user supplied parameters and Onshape will automatically create a usable UI for our feature, where the designer can input his parameters.

To create a parameter place the cursor in the precondition scope, click the "Feature parameters" button in the top bar (*** photo) and choose the appropriate type for your parameter.
You'll be given this parameter definition - 

```
annotation { "Name" : "My Count" }
        isInteger(definition.myCount, POSITIVE_COUNT_BOUNDS);
```

![](https://i.imgur.com/Rffx6mR.png)


You can replace ```"myCount"``` with a proper name for your parameter and the `annotation` "Name" with a user-visible version of this name.

The bounds argument specifies the possible range of values for this parameter - the default `POSITIVE_COUNT_BOUNDS` means any positive number can be used but custom bounds can be defined.

I'll demonstrate the `"Fret Count"` parameter (which specifies how many frets slots will be cut from the board) and it's bounds - 

```
// Defined in the global scope
export const FRET_BOUNDS =  {
    (unitless): [1, 24, 100]
    } as IntegerBoundSpec;

// Inside the feature precondition - 
annotation { "Name" : "Fret Count"}
isInteger(definition.fretCount, FRET_BOUNDS);
```

In the bound definition I define the possible bounds in every relevant unit of measurement. In this case I use `unitless` because this is an absolute number.
The bounds are the first and last members of the array (1 and 100) and the default value is the middle member (24 in my case)

![](https://i.imgur.com/lRslFHX.png)


Here's an example for a length parameter, which will be used to as a dimension in my geomtry - 

```
export const FRETBOARD_RADIUS_BOUNDS = {
    (inch): [10, 16, 25],
    (millimeter): 400,
    (centimeter): 40,
    (meter): 0.4,
    } as LengthBoundSpec;

annotation { "Name" : "Fretboard Radius"}
isLength(definition.fretboardRadius, FRETBOARD_RADIUS_BOUNDS);
```

Here I defined default values for multiple units of measurements which may be used by the designer, and I can set the bounds in any one of them and Onshape will automatically evaluate them, no matter which dimension was used.

Another fun pattern to use is conditional parameters, which will be visible for the designer only if he ticks a box, so the UI is less messy.
I define a `boolean` parameter, and if it's set to true I define some more features and a special `annotation` that makes them driven by the `boolean` feature. 

```
annotation { "Name" : "Spacing Options" }
definition.spacingOptions is boolean;

if (definition.spacingOptions) {
    annotation { "Group Name" : "Spacing Options", "Collapsed By Default" : true, "Driving Parameter" : "spacingOptions" } {
        annotation { "Name" : "Nut String Spacing"}
        isLength(definition.stringSpacing, STRING_SPACING_BOUNDS);
        
        annotation { "Name" : "Bridge String Spacing"}
        isLength(definition.bridgeStringSpacing, BRIDGE_STRING_SPACING_BOUNDS);
        
        annotation { "Name" : "String Edge Spacing"}
        isLength(definition.edgeSpacing, EDGE_SPACING_BOUNDS);
    }
}
```

![](https://i.imgur.com/XT5RNHi.png)


Onshape let's you define more types other the number and dimensions, such as strings, images, angles etc... but I didn't need any of them for this project.

You can see more examples of parameters in my code under the precondition scope.

# Sketching
Just like in standard CAD applications, to create 3D bodies we need to create 2D sketches first.
Automating sketches is very non-intuitive, as it requires you to describe every point, line or arc you use without using sketch relations such as "parallel" or "equal".
Let's move to the "function action" scope of our feature and see some examples -

```
var fretboardWidth = ((definition.stringCount - 1) * definition.stringSpacing + definition.edgeSpacing * 2);
var halfFretboardWidth = fretboardWidth / 2;

var fretboardBlankTop = newSketch(context, id + "fretboardBlankTop", {
        "sketchPlane" : qCreatedBy(makeId("Front"), EntityType.FACE)
});

skRectangle(fretboardBlankTop, "fretboardTopProfile", {
        "firstCorner" : vector(-halfFretboardWidth, 0 * millimeter),
        "secondCorner" : vector(halfFretboardWidth, -definition.fretboardThicknes)
});

skSolve(fretboardBlankTop);
```

There's quite a lot going on here, but all we did here was using some parameters we received to sketch a rectangle.
 - I first defined some dimensions variables by using the `var` keyword.
 - I access given paramters by using the definition map.
 - I can use basic math operations ([Or advanced ones](https://cad.onshape.com/FsDoc/library.html#module-math.fs)) to calculate the needed dimensions for my rectangle.

Creating a sketch is done using newSketch. 
We need to feed it a unique `ID` for our sketch, which is under the given feature `ID` in our hierarchy so something like `id + "SketchName"` works.

`"sketchPlane" : qCreatedBy(makeId("Front"), EntityType.FACE)` Means the sketch will be drawn on the default "Front" plane of the part. This exposes us to an important principle of FeatureScript -

### Queries - 
Queries allows us to access geometry (vertices, edges, faces, and bodies) by using a disgusting query syntax.

Sometimes we won't need this - if we want a reference for a geomtry we created, like entities in this very sketch we are creating, we can just keep them in a variable and point to them when they are used.

However, here we need to access a plane we did not create, so we make a query - `qCreatedBy(makeId("Front"), EntityType.FACE`), or in human language - "*Give me any entity of type FACE that was created by an operation with id == "Front"*".

This was the easiest way there is to grab the "front" plane, I swear.

There's a pleathora of other query functions used to fetch entities in all sorts of ways, like [`qSketchRegion`](https://cad.onshape.com/FsDoc/library.html#qSketchRegion-Id-boolean).

[This](https://cad.onshape.com/FsDoc/library.html#module-query.fs) section of documentation further describes this in further vague details.
<br>
Let's continue with our sketch.
```
skRectangle(fretboardBlankTop, "fretboardTopProfile", {
        "firstCorner" : vector(-halfFretboardWidth, 0 * millimeter),
        "secondCorner" : vector(halfFretboardWidth, -definition.fretboardThicknes)
});

skSolve(fretboardBlankTop);
```
We have functions with the prefix "sk" that let's us add entities to our sketch, like the `skRectangle` demonstrated above, which needs a sketch context argument, a sketch ID, and two points on the sketch to create a rectangle between.

The points are described as a `vector`which expects two dimensions to be defined.
Notice how I multiplied `0` by `millimeter` to create a dimension of 0mm as Onshape can't treat an integer (an abstract number) as a dimension of a distance.

The `skSolve` function solves the given sketch object, which adds it to the feature context, so we finally have something visible for the designer!

![](https://i.imgur.com/4PbIqzL.png)


Here's a nicer example of a sketch, which demonstrates the real power of automation for this use case - 

```
var fret_locations = makeArray(definition.fretCount + 1, 0);
var current_fret = definition.scaleLength;
for (var fret_num = 1; fret_num <= definition.fretCount; fret_num += 1) {
    current_fret = current_fret / (2 ^ (1/12));
    fret_locations[fret_num] = definition.scaleLength - current_fret;            
}

var fretSlots = newSketch(context, id + "fretSlotsProfile", {
    "sketchPlane" : qCreatedBy(makeId("Right"), EntityType.FACE)
});

for (var fret_num = 1; fret_num <= definition.fretCount; fret_num += 1) {
    skRectangle(fretSlots, "fretSlotsProfile"  ~ fret_num, {
            "firstCorner" : vector(fret_locations[fret_num] + definition.fretSlotWidth / 2, 0 * millimeter),
            "secondCorner" : vector(fret_locations[fret_num] - definition.fretSlotWidth / 2, -definition.fretSlotDepth)
    });
}

skSolve(fretSlots);
```

Here I use a some "math" to define dimensions for a sketch that contains multiple rectangle, and a `for` loop to create them.

Manually creating this in Onshape would've took me like 10 minutes of pointing and clicking, but I managed to create this automated sketch in only 40 minutes!

![](https://i.imgur.com/Rloypd1.png)

#  Going 3D

We can also automate extrude, revolve, loft etc... operations that use the sketches we defined to create 3D bodies.

These kind of operations are done by functions with "op" prefix - 
```
opLoft(context, id + "fretboardBlank", {
        "profileSubqueries" : [ qSketchRegion(id + "fretboardBlankTop"), qSketchRegion(id + "fretboardBlankBottom") ],
        "bodyType" : ToolBodyType.SOLID
});
```

Take this loft operation for example - it requires two sketches that define the profiles of the loft (and can be further customized with loft guides etc...).
To satisfy the `"profileSubqueries"` argument we will give it an array of queries.

The most suitable query for this case is `qSketchRegion` which takes a sketch `id` and finds the enclosed 2D regions this sketch creates.

![](https://i.imgur.com/yssHaCb.png)

 - TIP: We can call `opDeleteBodies` to get rid of any "construction geometry" that the designer don't need to see in his feature tree - 
```
//  We don't need "fretboardBlankTop" anymore so let's delete it
opDeleteBodies(context, id + "deleteSketchFretboardBlankTop", { "entities" : qCreatedBy(id + "fretboardBlankTop") });
```
<br>

You can also consider this example, a relatively complex extrude operation, which demonstrate usage of many non-default extrusion parameters.
```
opExtrude(context, id + "fretSlots", {
    "operationType": NewBodyOperationType.REMOVE,
    "entities" : qSketchRegion(id + "fretSlotsProfile"),
    "direction" : evOwnerSketchPlane(context, { "entity" : qSketchRegion(id + "fretSlotsProfile", false) }).normal,
    "endBound" : BoundingType.UP_TO_NEXT,
    "isStartBoundOpposite" : true,
    "endTranslationalOffset" : -definition.blindSlotThickness,
    "startBound" : BoundingType.UP_TO_NEXT,
    "startTranslationalOffset" : -definition.blindSlotThickness
});
```

Manually selecting each entity for this extrusion is very annoying and FeatureScript gets this done nicely with a query.

Another nice example is this boolean operation between 3D bodies, that also demonstrates nestled queries to filter query results - 
```
opBoolean(context, id + "fretSlotsCut", {
        "tools" : qBodyType(qCreatedBy(id + "fretSlots", EntityType.BODY), BodyType.SOLID),
        "targets": qBodyType(qCreatedBy(id + "fretboardBlank", EntityType.BODY), BodyType.SOLID),
        "operationType" : BooleanOperationType.SUBTRACTION
});
```
*"Get every body of type SOLID created by the "fretSlots" Operation."*

# Off to work
The full extent of the [FeatureScript Standard Library](https://cad.onshape.com/FsDoc/library.html) is vast.
Almost anything that can be done manually also has an API.
```
setProperty(context, {
    "entities" : qBodyType(qCreatedBy(id + "fretboardBlank", EntityType.BODY), BodyType.SOLID),
    "propertyType" : PropertyType.APPEARANCE,
    "value" : color(0.21, 0.21, 0.21)
});
```
This snippet will change the display color of my part :)

You can also do error handling - 
```
if (definition.stringCount < 2) {
    throw { message : ErrorStringEnum.INVALID_INPUT };
}
```
Some recommended resources that helped me navigating:
 - [This repo](https://github.com/dcowden/featurescript) has some references for nice features you can take examples from.
 - [This one](https://github.com/javawizard/onshape-std-library-mirror) is a copy of the FeatureScript standard library, which can be helpful when navigating unclear documentation and obscure errors.
 - The surprisingly active [Onshape forum](https://forum.onshape.com/).

# Auto-designing a fretboard
I thought it will be nice to actually describe the method I used to produce the fretboard.
Let's start with my input parameters - 

**General Options:**
 - Fret count
 - Scale length - affects the length of the fretboard
 - String count - affects the width of the fretboard
 - Radius - of the top curve

**Spacing Options**
 - Nut string spacing - distance between the strings at the nut
 - Bridge string spacing - distance between the strings at the bridge.
 - Edge spacing - distance between the outer strings and the edges of the fretboard
 
 **Advanced Options**
  - Blind slots - how far from the edge of the board the slot will end
  - Thickness - of the board
  - Body overlap - amount of extra material after the last fret

All of these are length parameters, besides the fret count, which is an integer.

I make the "fretboard blank" by creating a loft between two sketches - one on each side of the fretboard.

![](https://i.imgur.com/h0fbf08.png)

![](https://i.imgur.com/wr6xB5m.png)

I calculate the position of each fret, and create a sketch with the profile of each fret slot in the right location.
![](https://i.imgur.com/KaV2bPd.png)

- For now the fretboard stretches all the way to the bridge, we will fix that later

I can now cut the fret slots.
The easiest way to do this in a single operation (and not fret_count * operation) was to create a solid body from each fret and than applying a boolean operation between the two bodies - 
 - Fret slots are not cut yet - they are a separate solid body that collides with the fretboard.

![](https://i.imgur.com/0O9BKCy.png)
 
 - Now the fret slots are cut by using `opBoolean` between the two bodies.
![](https://i.imgur.com/rheVQ5o.png)

After cutting the excess material on the side, I draw a circle of the chosen radius on the side of the fretboard:

![](https://i.imgur.com/YMZd9ru.png)

And now I can cut the radius from the board with `opExtrude` - 

![](https://i.imgur.com/nCZGIYM.png)
<br><br><br>
<br>
and...
<br><br><br>
That's it!
A guitar fretboard design, fully automated.
![](https://i.imgur.com/UziedT5.gif)