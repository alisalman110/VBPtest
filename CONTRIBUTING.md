# Contributing

Make sure there is an outstanding issue relevant to the topic. Create an issue if necessary. Using issues allows for defining the scope before spending time on the content, discuss tht content with the peers to shape it further, and benefit from advanced branching and workflow features.

Take the ownership of the issue by assigning it to yourself.

Create a merge request for the issue. Creating merge request will additionally create a new branch dedicated for this specific task. Branching out for each piece of content helps with isolating work and easier merge to upstream if the content requires updating different sections of the guide. Merge requests help with the peer review when the content is ready.

Work on the content - write new or update or validate depending on the nature of the task. Use the corresponding branch.

When done go to merge request and resolve WIP status. It will indicate that the work is done and the merge request can now be merged upstream.

Now the content can be reviewed by the team and discussed in merge request comments or merged to become a part of the main content.

Rinse and repeat.

## File & Folder Structure

For a TOC like this

* Anatomy
    * vSphere
        * Interaction
        * Transport Modes            
    * Hyper-V
        * Interaction
    * Backups 

The folder structure should look like this

    anatomy/
    |-- vsphere/
    |   |-- interaction.md
    |   |-- transport_modes.md
    |   |-- media/
    |       |-- transport_mode_whatever_is_shown.png
    |       |-- transport_mode_image_name2.png        
    |       |-- interaction_image_name.png
    |-- hyperv/
    |   |-- interaction.md
    |-- backups.md
    |-- media/
        |-- backups_321.png
     
## Page Headers

The BP will be based on a [Just-The-Docs](https://pmarsceill.github.io/just-the-docs/) 
template, so every page needs a specific header to be automatically included in
the ToC and navigation. Check out [Navigtion Structure](https://pmarsceill.github.io/just-the-docs/docs/navigation-structure/)
section for details.

### Examples

Some examples which can be copy & pasted and adapted:

**Root Section** (`/anatomy/`)

    ---
    title: Anatomy
    nav_order: 20
    has_toc: false
    has_children: true
    permalink: /anatomy/
    ---

**Page in root-section** (`/anatomy/interaction.md`)

    ---
    title: Guest Interaction
    parent: Anatomy
    nav_order: 60
    permalink: /anatomy/guestinteraction/
    ---
    
**Sub-Section** (`/anatomy/hyper-v/index.md`)

    ---
    title: Hyper-V
    parent: Anatomy
    nav_order: 30
    has_toc: false
    has_children: true
    permalink: /anatomy/hyper-v/
    ---
    
**Page in sub-section** (`/anatomy/hyper-v/instantvmrecovery.md`)

    ---
    title: Instant VM Recovery
    parent: Hyper-V
    grand_parent: Anatomy
    nav_order: 60
    ---
