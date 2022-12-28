As a developer, understanding the fundamentals of the design tools used within your team is an essential skill. The notorious war between
developers and designers is never-ending and design tools are the places where these two worlds collide. This raises the importance of using
good design tools which provide a nice connection between the design and development process. Imagine if all the screen designs, components,
different assets were stored on Google Drive and the review process and communication was handled through email/Slack. To get a single
component, you'll have to go through several folders and then, you'll have to open your email/Slack to remind yourself about the discussions
you had regarding the implementation. This isn't a big deal if you have to do it once in a day, however, this is a continuous and frequent
process. Eventually, the project grows, new designs are requested, new folders and files are added and all of a sudden, handling designs
like this sounds like a nightmare, right?

![Designers vs Developers meme](/img/figma/figma_designers_vs_developers_meme.png "Designers vs Developers meme")

[Figma](https://www.figma.com) is a cloud-based design tool which provides simple, effective and intuitive functionalities perfect for team
collaboration. Similar to Google Docs, team members can easily communicate, collaborate and edit the same file in Figma in real time.

## Benefits of using Figma

- **Simple and intuitive**

Most of us are already very familiar with using tools like Google Docs and Figma isn't much different from that. First-time Figma users can
get quickly adjusted to using Figma and it won't take them too long to understand most of its functionalities.

- **Easier team collaboration and communication**

Communication and collaboration within Figma is extremely easy and convenient since Figma allows you to leave comments on the designs.
Discussions can be opened in the comment's thread and if you have the right permissions for it, you could also @mention certain people in
order to get a faster response from the discussion-relevant team members.

- **Everything you need is at one place**

Screen designs, components, assets, styles, comments, discussions and so much more can be found within a single Figma file. No need to
include any third-party tools (unless the team feels like they would benefit from them), so basically, for all UI related stuff, you can
refer to the Figma file. You want it, you open Figma, you got it, no further actions needed. :)

- **Platform flexibility**

If you have any operating system and you run any web browser, you are ready to use Figma. You can share, open, edit Figma files through your
web browser without any additional setup. The universality of Figma means that the design work will be available to everyone, despite the
platform they are using.

- **Flexible user permissions management**

Each person can have either a view or an edit access to the Figma files. Additionally, the admin and owner roles can be set accordingly. All
permissions can be easily managed and customized as desired. For a more detailed description about Figma team permissions and what different
access modes entail, check out the following [link](https://help.figma.com/hc/en-us/articles/360039970673-Team-permissions).

## Figma essentials

Let's take a look at some of the things which you'll probably be using very frequently as a developer.

### Frames

![Figma Frames](/img/figma/figma_frames.png "Frames in Figma")

Frames serve as top-level containers for most elements in Figma. Usually, each screen has a separate frame and they are all listed
on the left side as shown in the image above. In this example project, the frames are located in the Layers tab, however, keep in
mind that this is customizable and the tab naming in your project might differ from this project's conventions (for example, you might have
a Pages tab rather than a Layers tab). What's important is to know the fundamentals and what Figma provides as functionalities and then,
minor things like naming conventions and way of structuring stuff within the team is easily manageable.

Each frame can be expanded and its sub-frames and elements will be shown as in the Register frame on the image above. You can click around
and explore what each frame consists of. You can select a particular frame or an element from the frame directly on the design canvas as
well.

### Element selection, code inspection, layout measurements and constraints

If you want to get more details about a certain element/component/part of the design, you can select it and then check out its details on the
right side of the Figma file.

Here's an example when the "NEXT" button from the Register screen is selected:

![Figma Element Inspection](/img/figma/figma_button_inspection.png "Element inspection in Figma")

In the Inspect tab on the right side, there is a Properties section which has some basic information about the dimensions of the selected
element. In the Code section, there is a small code snippet which can be used as a guideline for implementing the design in code. On this
image, Android is selected as the preferred platform in the drop-down menu, however, you can choose iOS or CSS and the code snippets will be
displayed accordingly. Depending on the selected element, more sections might be present in this Inspect tab, such as sections with
information about Colors, Borders, Typography, etc. If you hover around the designs and have the button selected, you can see some lines
which represent its constraints. You can also view the margins, spacing and other dimensions by hovering over particular regions in the
designs.

### Exports

In the Figma files for the project you'll be working on, there will probably be an Export tab on the right side as well when a certain
element/component is selected. If you have edit access, you'll have the Export section in the Design tab, however, for developers, it is
usually enough to have only view access to the Figma files.

The Export tab will look something like this (there are also images from several options provided in this tab which will be discussed
below):

![Exporting in Figma](/img/figma/figma_exports.png "Exporting in Figma")

In the Export tab, the default export size is 1x and when you click on it, a drop-down list with more options is displayed. You can also
write the suffix which you would like the exported file to have and choose the image format of the export. Additionally, there is an option
to set some extra settings if you click on the three dots placed after the image format. If you click on the "+" button, you can export
multiple files at once (you can set different settings for each of the exported files). The Preview section lets you see how the exported
file will look like after the export. Here's
a [link](https://help.figma.com/hc/en-us/articles/360040028114-Guide-to-exports-in-Figma) to a nice guide for exports in
Figma.

### Comments and discussions

You can enter Comment mode by clicking on the Comment icon in the toolbar. As long as you are in Comment mode, you can add a comment
anywhere on the canvas by clicking on the spot where you want it to be added.

![Comments in Figma](/img/figma/figma_comment.png "Comments in Figma")

Other team members can join in the discussion by replying to the comment and as it was mentioned before, you can @mention other team members
in the comments. You can also resolve the comment thread once the discussion is done.

![Replying and resolving comments in Figma](/img/figma/figma_comment_resolve.png "Replying and resolving comments in Figma")

When a comment is resolved, by default, it will no longer appear on the canvas. However, you can customize this and some other settings in
the options menu shown in the image below:

![Figma comment settings](/img/figma/figma_comment_settings.png "Comment settings in Figma")


## Useful links

- [From Zeplin to Figma](https://infinum.com/blog/zeplin-to-figma)
- [Infinum Design Handbook](https://infinum.com/handbook/design)


![Figma meme](/img/figma/figma_lion_king_meme.jpg "Figma meme")
