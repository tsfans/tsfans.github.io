---
title: eclipse application
toc: true
date: 2022-05-13 08:45:58
tags: eclipse
categories: ide
---

## 1.项目结构

### application model

This model contains the parts of the application as individual model elements and their hierarchical relationship.

- windows
- parts
  - views
  - editors
- menus
- toolbars
- handlers
- commands
- key bindings

- has attributes which describe its current state
- contains links to Java classes or static resources
  - java classes: bundleclass://test/test.parts.MySavePart
  - static resources: platform:/plugin/com.example.plugin/icons/save_edit.gif
- The application model is extensible, e.g., other plug-ins can contribute to it via model processors and model fragments.

### overview of the model objects

- application: MApplication
  - Describes the application object. All other model elements are contained in this object.
- window
  - MWindow
  - MTrimmedWindow: can contain trimbars (which can hold toolbars).
- parts: MPart
  -  Parts can be stacked or positioned next to each other depending on the container into which they are dropped.
  -  A part can have a drop-down menu, context menus and a toolbar.
  -  Parts can be classified as views and editors.
     -  views: display and modify a set of data. change data don't need to explicitly save the editor content
     -  editors: modify a single data element. change data has to explicitly save the editor content
- parts containers
  - Part Stack: A part stack arranges its children similar to a browser
  - Part Sash Container: displays all its children at the same time either horizontally or vertically aligned.
- perspective
  - A perspective is an optional container for other part containers and parts.
  - It is presented by MPerspective objects, which must be placed in an MPerspectiveStack object.
  - EPartService: Switching perspectives
- MAddon: A self-contained component typically without user interface. It can register for events in the application life cycle and handle these events.
- MDirtyable
  - Property of MPart which can be injected. If set to true, this property informs the Eclipse platform that this Part contains unsaved data (is dirty)
  - In a handler you can query this property to provide a save possibility.
- MPartDescriptor
  - a template for new parts. 
  - A new part based on this part descriptor can be created and shown via the Eclipse framework.
- Snippets
  - Snippets can be used to pre-configure model parts which you want to create via your program. 

### Extending the application model from other plug-ins

- contributions
  - Static contributions: text files(fragments or model fragments.)
  - Dynamic contributions: java classes(processors or model processors.)

- model fragments
  - A model fragment is a file which typically ends with the .e4xmi extension.