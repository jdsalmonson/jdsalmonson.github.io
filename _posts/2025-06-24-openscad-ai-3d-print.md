---
layout: post
title: Using AI Prompts to Create STL Files for 3D Printing
categories: [CAD, 3D Printing, Video]
image: /assets/images/openscad_ai/button_three_sections.png
---


<div style="display: flex; justify-content: center; align-items: center;">
    <iframe 
        width="315" 
        height="560" 
        src="https://www.youtube.com/embed/eCS01HnPdSs"
        frameborder="0" 
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
        allowfullscreen>
    </iframe>
</div>

It recently occurred to me that it should be possible to use an AI prompt to describe what I want to 3D print without using a CAD program.  While this wouldn't be a feasible solution for an elaborate CAD creation, it can be useful for quick one-off objects or prototypes.  My first idea was to use an AI agent, in this case Claude Sonnet 3.5, to generate Python code to create the STL code for the 3D printer using the [`numpy-stl`](https://pypi.org/project/numpy-stl/) package.  Interestingly, this attempt failed pretty quickly.  While the `numpy-stl` package works well at turning a mesh into an STL file, I found that the AI agent's ability to prescribe a consistent mesh was inadequate, so strange, inconsistent polygon structures would be generated that were unable to be printed.  So it occurred to me that it might be possible to use [OpenSCAD](https://openscad.org/), the open-sourced programmable 3D CAD modeler.  OpenSCAD has its own programming language for creating CAD models and is very good at generating usable STL files from this code.  It is a much simpler task for an AI agent to create the OpenSCAD code from a prompt.


My first attempt at this was to create a button from my wife's coat, as illustrated in the video above.  For reference, the OpenSCAD code for this button is available [below](#button-openscad-code).  I constructed the radial surface profile of this button from three sections of circular arc.  When I specified how I wanted these sections of arc to connect together, much like an algebra homework problem, the agent did a poor job of properly interpreting what I was asking for.  I know these agents are able to answer algebra word problems, so I'm not sure why it wasn't doing so in this case.  Ultimately, I had to do the algebra myself to give it the proper parameters, which worked fine, but was more involved than I'd hoped.  

Since making this button, I've created a half-dozen objects to 3D print using this tecnique and I've been pretty happy with it.  One benefit of this technique is that the result is a readable, parameterized piece of code that can be easily scaled or altered by hand after the fact.

<div style="display: flex; justify-content: center; align-items: center; gap: 20px; margin: 20px 0;">
    <img src="/assets/images/openscad_ai/button_three_sections.png" alt="OpenSCAD output" style="max-width: 45%; height: auto;">
    <img src="/assets/images/openscad_ai/button_three_sections_stl.png" alt="STL file" style="max-width: 45%; height: auto;">
</div>
<p style="text-align: center; font-style: italic; margin-top: 10px;">OpenSCAD output for button design (left) and resulting STL file (right).</p>

---
## Example: Make a Drink Coaster

Here is a complete simple example of how to generate a drink coaster with a single AI prompt (using [Cursor](https://www.cursor.com/)).  First, I asked the AI agent for a coaster with the following prompt:

<blockquote style="font-family: monospace; white-space: pre-wrap; word-wrap: break-word; background-color: #f6f8fa; padding: 16px; border-left: 4px solid #0366d6; margin: 16px 0;">Generate the openscad code to create a square coaster, 6 cm on a side, with a 4 mm thickness.  Round the four corners with a radius of curvature of 0.5 cm.  On the face of the coaster, add four cylindrical pads, each at the corner of a square centered on the original square with 4 cm on a side.  Each of these cylinders have a radius of 3 mm and an extruded height above the coaster surface of 2 mm.
</blockquote>

The AI agent then produced this OpenSCAD code which I saved to a file `coaster.scad`:
```cpp
// Square Coaster with Rounded Corners and Cylindrical Pads
// Dimensions: 6cm x 6cm x 4mm with 0.5cm corner radius
// Four cylindrical pads: 3mm radius, 2mm height, positioned at corners of 4cm square

// Main coaster dimensions
coaster_size = 60; // 6cm in mm
coaster_thickness = 4; // 4mm
corner_radius = 5; // 0.5cm in mm

// Pad dimensions and positioning
pad_radius = 3; // 3mm radius
pad_height = 2; // 2mm height above surface
pad_square_size = 40; // 4cm square for pad positioning

// Calculate pad positions (corners of 4cm square centered on 6cm coaster)
pad_offset = pad_square_size / 2; // 20mm from center

// Main coaster body with rounded corners
module coaster_body() {
    hull() {
        // Create rounded corners using cylinders
        for (x = [-1, 1]) {
            for (y = [-1, 1]) {
                translate([x * (coaster_size/2 - corner_radius), 
                          y * (coaster_size/2 - corner_radius), 
                          coaster_thickness/2]) {
                    cylinder(h = coaster_thickness, r = corner_radius, center = true);
                }
            }
        }
    }
}

// Individual cylindrical pad
module pad() {
    cylinder(h = pad_height, r = pad_radius, center = false);
}

// All four pads positioned at corners of 4cm square
module pads() {
    for (x = [-1, 1]) {
        for (y = [-1, 1]) {
            translate([x * pad_offset, y * pad_offset, coaster_thickness]) {
                pad();
            }
        }
    }
}

// Complete coaster assembly
module coaster() {
    coaster_body();
    pads();
}

// Generate the coaster
coaster();
```

I then ran this file from the command-line with [OpenSCAD](https://openscad.org/) from which I could create an STL file, shown below:
```bash
openscad coaster.scad
```

<div style="display: flex; justify-content: center; align-items: center; gap: 20px; margin: 20px 0;">
    <img src="/assets/images/openscad_ai/coaster_openscad.png" alt="OpenSCAD output" style="max-width: 45%; height: auto;">
    <img src="/assets/images/openscad_ai/coaster_stl.png" alt="STL file" style="max-width: 45%; height: auto;">
</div>
<p style="text-align: center; font-style: italic; margin-top: 10px;">OpenSCAD output from <code>coaster.scad</code> (left) and resulting STL file (right).  Notice the discrete polygons are plainly visible in this model.  I will rectify that below.</p>

Typically, I find this design cycle to be iterative; progressively asking for more elaboration.  In this case, I'd like to add more polygons to the rounded edges to make them appear smoother.  Asking the AI agent how to do that yielded the solution: add the argument, `$fn = 64`, to the `cylinder(...)` function:


```diff
// Main coaster body with rounded corners
module coaster_body() {
    hull() {
        // Create rounded corners using cylinders
        for (x = [-1, 1]) {
            for (y = [-1, 1]) {
                translate([x * (coaster_size/2 - corner_radius), 
                          y * (coaster_size/2 - corner_radius), 
                          coaster_thickness/2]) {
-                    cylinder(h = coaster_thickness, r = corner_radius, center = true);
+                    cylinder(h = coaster_thickness, r = corner_radius, center = true, $fn = 64);
                }
            }
        }
    }
}

// Individual cylindrical pad
module pad() {
-    cylinder(h = pad_height, r = pad_radius, center = false);
+    cylinder(h = pad_height, r = pad_radius, center = false, $fn = 64);
}
```

<div style="display: flex; justify-content: center; align-items: center; gap: 20px; margin: 20px 0;">
    <img src="/assets/images/openscad_ai/coaster_hi_openscad.png" alt="OpenSCAD output" style="max-width: 45%; height: auto;">
    <img src="/assets/images/openscad_ai/coaster_hi_stl.png" alt="STL file" style="max-width: 45%; height: auto;">
</div>
<p style="text-align: center; font-style: italic; margin-top: 10px;">OpenSCAD output with more polygons for a smoother appearance (left) and resulting STL file (right).</p>

Note that OpenSCAD automatically re-reads and renders the input file, `coaster.scad`, when it has been changed which allows for quick design iteration.  Also, the STL file (in binary format in this case) can be directly generated from the command-line:
```bash
openscad --export-format binstl -o coaster.stl coaster.scad
```
and the resulting `coaster.stl` file is ready to be sent to a 3D printer.

---
## Future Directions

Another option I'm interested to try is to use this AI generated OpenSCAD code as a starting point for a more traditional CAD program.  The relative ease of quickly creating a basic model from an AI prompt can save time setting up a base design that can be built on with an interactive CAD interface.  It appears that [FreeCAD](https://www.freecad.org/) can import the `*.scad` file which can then be altered or exported in STEP format which is readable by other CAD programs.


---

## Button OpenSCAD Code {#button-openscad-code}

```cpp
// Parameters
button_diameter = 23; // Diameter of the button in mm
button_height = 3.5;    // Height of the button in mm
hole_diameter = 2;    // Diameter of the buttonholes in mm
hole_spacing = 3.5;     // Edge length of the square arrangement of holes in mm

// Derived parameters
center_height = 0.4*button_height; // Height at center
cord_radius = 7;     // Radius where first and second sections meet
cord_height = 0.6 * button_height; // Height at cord_radius
edge_radius = button_diameter/2; // Radius at the edge
outer_radius = 0.9 * edge_radius; // Radius where second and third sections meet
outer_height = button_height; // Height at outer_radius
edge_height = 0.9 * outer_height; // Height at the edge

// Function to calculate circle radius
function get_circle_radius(cord_radius, cord_height, center_height) = 
    (pow(cord_radius, 2) + pow(cord_height - center_height, 2)) / 
    (2 * (cord_height - center_height));

// Function to calculate height for section 1 (concave)
function section1_height(r) =
    let(
        R = get_circle_radius(cord_radius, cord_height, center_height),
        // Height is center_height + R - sqrt(R^2 - r^2)
        // This ensures h = center_height when r = 0
        // and h increases with r
        h = center_height + R - sqrt(pow(R, 2) - pow(r, 2))
    )
    h;

// Function to calculate height for section 2 (convex)
function section2_height(r) =
    let(
        // For a circle with center at (outer_radius, h_C)
        // that passes through (cord_radius, cord_height) and (outer_radius, outer_height)
        // and has zero slope at outer_radius
        // We can solve for h_C:
        h_C = (pow(outer_radius - cord_radius, 2) + pow(cord_height, 2) - pow(outer_height, 2)) / (2 * (cord_height - outer_height)),
        // Then R = outer_height - h_C
        R = outer_height - h_C,
        // Center is at (outer_radius, h_C)
        center_x = outer_radius,
        center_y = h_C,
        // Height is center_y + sqrt(R² - (r - center_x)²)
        h = center_y + sqrt(pow(R, 2) - pow(r - center_x, 2))
    )
    h;

// Function to calculate height for section 3 (convex)
function section3_height(r) =
    let(
        // For a circle with center at (outer_radius, y0)
        // that passes through (outer_radius, outer_height) and (edge_radius, edge_height)
        // and has zero slope at outer_radius
        // The center must be at (outer_radius, outer_height - R)
        // where R is the radius of the circle
        // We can solve for R using the edge point:
        // (edge_radius - outer_radius)² + (edge_height - (outer_height - R))² = R²
        // This gives us:
        R = (pow(edge_radius - outer_radius, 2) + pow(edge_height - outer_height, 2)) / (2 * (outer_height - edge_height)),
        // Center is at (outer_radius, outer_height - R)
        center_x = outer_radius,
        center_y = outer_height - R,
        // Height is center_y + sqrt(R² - (r - center_x)²)
        h = center_y + sqrt(pow(R, 2) - pow(r - center_x, 2))
    )
    h;

// Function to get height at any radius
function get_height(r) =
    r <= cord_radius ? section1_height(r) :
    r <= outer_radius ? section2_height(r) :
    r <= edge_radius ? section3_height(r) :
    edge_height;

// Generate points for the profile
num_points = 100;
profile_points = [
    for (i = [0:num_points])
    let(
        r = i * edge_radius / num_points,
        h = get_height(r)
    )
    [r, h]
];

// Create the complete profile
// Start with the top profile
top_profile = profile_points;

// Create the bottom profile
bottom_profile = [
    [edge_radius, 0],  // Start at the edge
    [0, 0]            // End at the center
];

// Combine all points to create a closed polygon
all_points = concat(
    top_profile,                     // Top profile from center to edge
    bottom_profile                   // Bottom profile from edge to center
);

// Create the 3D button with holes
difference() {
    // Create the button by rotating the profile
    rotate_extrude($fn = 100) {
        polygon(all_points);
    }

    // Create the four buttonholes
    for (x = [-hole_spacing/2, hole_spacing/2])
        for (y = [-hole_spacing/2, hole_spacing/2])
            translate([x, y, 0])
                cylinder(d = hole_diameter, h = button_height + 1, center = true, $fn = 100);
} 
```