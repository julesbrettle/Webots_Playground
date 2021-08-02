# Webots_3D_Flippy_Robots (Formerly Webots_Playground)

### Table of Contents

1. [Project Purpose and Progress](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#1--project-purpose-and-progress)
2. [Environment Setup & Quick Start](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#2--environment-setup--quick-start)
3. [File Structure](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#3--file-structure)
    - [`./controllers` Directory](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#controllers-directory)
    - [`./plugins` Directory](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#plugins-directory)
    - [`./protos` Directory](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#protos-directory)
    - [`./script_inputs` Directory](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#script_inputs-directory)
    - [`./script_outputs` Directory](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#script_outputs-directory)
4. [Useful Resources & References](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#4--useful-resources--references)
    - [This Repo's Wiki](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#this-repos-wiki)
    - [Creating a Physics Plug-in with ODE](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#creating-a-physics-plug-in-with-ode)
    - [Webots General Documentation](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#creating-a-physics-plug-in-with-ode)
    - [Conceptual](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#conceptual)
5. [Recommendations and Next Steps](https://github.com/julesbrettle/Webots_3D_Flippy_Robots/blob/main/README.md#5--recommendations--next-steps)

---

## 1 | Project Purpose and Progress
*Section created by J. Brettle (Summer 2021)*
*Section modified by __*

This project builds upon the research done by [Lucie Houel](https://ssr.seas.harvard.edu/files/ssr/files/phdthesis2020malley.pdf), Melinda [Malley](https://ssr.seas.harvard.edu/files/ssr/files/phdthesis2020malley.pdf), and others in using "flippy" robots (modeled as two spheres that flip around each other to walk forwards and over each other) to mimic the self-assembly behaviors of army ants. Most of their simulations were done in  2D, so this project aims to create a framework for modeling similar behaviors and rulesets in 3D using the open source environment Webots. 

As of 8/2/2021, the flippies can relatively reliably walk on the floor and each other. They use their four touch sensors to differentiate between touching another flippy and the floor, and change their behavior as such (see "The 4 touch sensors theory" section). They have the ability to change heading and navigate across the floor using two extra motors for turning, as well as a GPS and Inertial Unit for guidance. Their current default mode is to turn to face and walk toward the first `goalCoords` point given (see "Flippy steering" section). The flippies also take longer between flips as they near the first given `slowCoords` point in order to create traffic to facilitate self-assembly (see "Flip delay / distance variable speed control" section). 

---

## 2 | Environment Setup & Quick Start
*Section created by J. Zerez (Spring 2021)*
*Section modified by J. Brettle (Summer 2021)*

**TL;DR: Download [Webots](https://cyberbotics.com/), Install a [C/C++ Compiler for Webots](https://cyberbotics.com/doc/guide/using-cpp) and make sure you have Python v3+. Then run a `.wbt` file from the worlds folder.**

The simulation is built off of [Webots](https://cyberbotics.com/), which is an open source robot simulator similar to gazebo. It support various different programming languages such as C, C++, Python, Java, and Matlab, as well as frameworks like ROS. The code for this project that interfaces directly with Webots is written in C++. As such, you'll need to make sure you have a C/C++ compiler. Instructions can be found [here](https://cyberbotics.com/doc/guide/using-cpp).

Additional python scripts have also been created in order to make the process of creating Webots simulations and worlds a bit more streamlined. Make sure you have some kind of Python version 3.

In Webots, one can choose to open and run [`flippy_sim.wbt`](/worlds/flippy_sim.wbt). It usually contains a singular flippy and can be useful for testing flippy parameters and geometries. Run [`flippy_sim_autogenerated.wbt`](/worlds/flippy_sim_autogenerated.wbt) for more flippies (see "Using flippy_sim_autogenerated and Script Inputs").

For best viewing, go to "Optional Rendering" under the "View" menu and select "Show Coordinate System" and "Show All Bounding Objects". "Show Contact Points" can also be a useful feature for debugging. 

---

## 3 | File Structure
*Section created by J. Zerez (Spring 2021)*
*Section modified by J. Brettle (Summer 2021)*

### `./controllers` Directory

The `./controllers` directory contains folders for each of the controller files. **The main controller for this project can be found in [`flippy_controller.cpp`](controllers/flippy_controller/flippy_controller.cpp)**. Changing this file is the key to modifying flippy behavior.

### `./plugins` Directory

This contains folders for various custom physics plugin files. **This project relies on a custom physics plugins which can be found in [`flippy_physics.cpp`](/plugins/physics/flippy_physics/flippy_physics.cpp)**. For more information about custom physics plugins, checkout [Webot's Documentation on physics plugins.](https://cyberbotics.com/doc/reference/physics-plugin)

### `./protos` Directory

This contains `.proto` files. These files are essentially wrappers for complex Webots objects. These files are used for specifying object parameters and architectures, giving the user a simplified set of parameters to interact with in the Webots UI. It substantially simplifies the feature tree of the project. [More information about protos can be found here](https://cyberbotics.com/doc/guide/tutorial-7-your-first-proto). This project does not use `.proto` files to define each flippy robot. This is because of the way that Webot's physics plugins are implemented. Each flippy in a swarm must be its own unique robot with a unique ID and DEF. As such, `.proto` files will not work. However, 4-touch-sensor flippies (light and dark tints of green and blue) use [`hemisphere.proto`](/protos/hemisphere.proto) to define the `indexedFaceSet` for the shape.

### `./script_inputs` Directory

This contains template files for the [`make_flippys.py`](/make_flippys.py) script. There are two required files:

- [`flippy_sim_template.wbt`](/script_inputs/flippy_sim_template.wbt): This file is the base world file. It is required to create an autogenerated world.
- [`flippy_template.proto`](/script_inputs/flippy_template.proto): This file is the base flippy file. IT is required to create each flippy in the swarm.

### `./script_outputs` Directory

This contains outputs from the [`make_flippys.py`](/make_flippys.py) script (if `write_to_wbt=False`). These files will be `.txt`'s and will be properly formatted to be pasted directly into any `.wbt` file.

---

## 4 | Useful Resources & References
*Section created by J. Zerez (Spring 2021)*
*Section modified by J. Brettle (Summer 2021)*

### This Repo's Wiki

- 

### Creating a Physics Plug-in with ODE

- [ODE's wiki style documentation](http://ode.org/wiki/index.php?title=Manual)
- [ODE's joint object documention (function list)](http://opende.sourceforge.net/docs/group__joints.html)
- [WeBots Physics Plugin example code (plugin)](https://github.com/cyberbotics/webots/blob/released/projects/samples/howto/plugins/physics/flying_mybot/flying_mybot.c)
- [Webots Physics Plugin example code (controller)](https://github.com/cyberbotics/webots/blob/released/projects/samples/howto/controllers/physics/physics.c)
- [Webots utility functions for interacting with ODE](https://www.cyberbotics.com/doc/reference/utility-functions)

### Webots General documentation

- [Webots Nodes and API Documentation](https://cyberbotics.com/doc/reference/nodes-and-api-functions?tab-language=c++). This resource contains links to documentation for the various Webots objects. This is useful to understand how they work and how to integrate them into your controller.
- [Webots Robot reference page](https://cyberbotics.com/doc/reference/robot?tab-language=c++)
- [Webots User Guide Tutorials](https://cyberbotics.com/doc/guide/tutorials)
- [Webots General FAQ](https://cyberbotics.com/doc/guide/general-faq)

### Conceptual

- [2D Flippy Sim Paper](https://ssr.seas.harvard.edu/files/ssr/files/phdthesis2020malley.pdf) (Houel)
- [Army Ant Self-Assembly Paper](https://ssr.seas.harvard.edu/files/ssr/files/phdthesis2020malley.pdf) (Malley)

---

## 5 | Recommendations & Next Steps
*Section created by J. Zerez (Spring 2021)*
*Section modified by J. Brettle (Summer 2021)*

- Implement flippy-to-flippy fixed joints (see "The usage of exclusively "flippy-to-floor" fixed joints as opposed to including "flippy-to-flippy" fixed joints" section).
- Do more debugging on the crashing problem (see "Individual robot crashing and running the simulation with multiple flippies" section).
- Try building a tower and/or a bridge. You may have to:
    - Create a way for flippies to switch to the next point in the `goalCoods` list when they are within the error margin (the fourth/last element in each row)
    - Create a way for flippies to use all of the points in `slowCoords` instead of just the first row
- Continue to adjust rulesets and parameters to more reliably create desired behaviors, and document effects of different changes.

---
