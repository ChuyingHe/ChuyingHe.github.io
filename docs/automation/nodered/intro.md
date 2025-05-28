NodeRed is NodeJS based low-code programming for event-driven applications. Can run on:

- Local
- Docker
- Devices: Raspberry Pi, FlowFuse Device Agent ...
- Cloud: AWS, Azure
...


!!! info "Docs"
    - [offcial doc](https://nodered.org/)
    - [Youtube Tutorial](https://www.youtube.com/watch?v=3AR432bguOY&list=PLKYvTRORAnx6a9ETvF95o35mykuysuOw)


Each **Node**:

- performs an individuell function
- self-contained: a.k.a NO external dependencies
- modula
- reusable


NodeRed uses **flow-based programming**, to start the flow, a msg need to be injected through a **Input Node**.


# Nodes

## Common Nodes

### Input Node
to inject messages

### Debug Node
Attention: You need to activate the Node by click on the panel.

You can log the whole `json` object and paste it directly where you needed it:

<img src="../imgs/json_object.png" width="600" />

### Function Node
the most powerful node, you can write custom JS code there.

### Exec Node
another VERY power node that can execute CLIs. 

#### JS Variables

|Variable|||
|:-|:-|:-|
|`context`|local variable ONLY for the current flow/line||
|`flow`|global variable for the current flow tab||
|`global`|global variable for ALL flow tab||
|``|||

### Link Nodes (Pair)
it has the following functionalities:

- Share data: it can pass Data through Flow in different tabs
- Triggers other flows

### Catch Node
to catch errors in the flow

## Install new Node
There are more available Nodes on the market, you can install them by:

<img src="../imgs/import_1.png" width="600">
<img src="../imgs/import_2.png" width="600">


# Flow
you can also import/export a **Flow**. You can do this for:

- selected Nodes
- current flow
- all flows

in the format of:

- Clipboard
- Library: like a template that can be reused later

## Subflow
to create a **Subflow**, select a node, go to **Setting** > **Subflows** > **Selection to Subflow**.

to edit the Subflow, double click > **Edit Subflow**


## Flow in Tab
double click on the tab to rename/delete the flow

# Shortcuts
You can find all the shortcuts under **Setting** > **Keyboard**


- `ctrl+Z` undo
- `ctrl+C` copy
- `ctrl+V` paste
- `ctrl+X` cut