---
{"dg-publish":true,"permalink":"/blogs/obisidan/publish-your-obsidian-vault-with-digital-garden/","tags":["blogs","obsidian"],"created":"2023-10-31T12:25:00.191+08:00","updated":"2023-10-31T21:05:29.732+08:00"}
---

# Unfolding the Digital Garden: A Fresh Approach to Deploying Your Obsidian Vault
I came in touch with the idea of digital garden recently and I feel that this is just what I needed to organise all my thoughts and learnings. Since then I switched from [[Blogs/Obisidan/Obsidian Learning\|notion to obsidian]] due to various reasons. After that I want to publish some of the things that i record online hoping that it could help someone in the future.

I have tried various methods, from [quartz](https://quartz.jzhao.xyz) to [obsidian publish](https://obsidian.md/publish) and now [digital-garden](https://obsidian.md/publish). I feel that this is the solution that I am looking for.

I am going to share some of my thoughts when I am using digital-garden
# Where the magic happens
Digital garden has a very detailed [documentation](https://dg-docs.ole.dev/) that really simplifies the setup process.
All you need is:
1. deploy a project in [vercel](https://github.com/oleeskild/digitalgarden) 
2. install the [digtial-garden obsidian pluging](https://github.com/oleeskild/obsidian-digital-garden)
3. add `dg-publish:true` to the properties of the notes that you want to publish
4. run the `publish` command for digital garden

Now you will have your site running for the world to view

# Whats the magic

## Themes
There are various things that you can configure about this site that you published, my favourite one will be the [theme setting](https://dg-docs.ole.dev/getting-started/04-appearance-settings/)!

You can change the color theme of you publish site using many predefined templates, when i as many, there is really **A LOT** of themes available for you to choose and I think there must be one that suits your taste. This is something that other publishing services is lacking.

# Compatibility
## Dataview
As of Nov 2023, there is literally 1.1 million users using [dataview](https://blacksmithgu.github.io/obsidian-dataview/)in their obsidian vaults to organise their notes. I have always wanted to show the notes that are retrieved by dataview in my digital garden, but it is not supported by other publish services. In [digital-garden](https://dg-docs.ole.dev/features#dataview-queries), dataview just works out of the box and it looks exactly the same as how it will appear in you obsidian vault.
### How it looks
![Pasted image 20231031202424.png](/img/user/Blogs/Obisidan/attachments/Pasted%20image%2020231031202424.png)

## Excalidraw
[Excalidraw](https://excalidraw.com) is an awesome open source white board tool that I can utilize to illustrate the concepts. Obsidian has an official excalidraw plugin that we can use to draw diagrams and embed it in our notes.

When i tried to use quartz to publish my notes previously, it is not compatible with excalidraw. I found a work around, which is to export excalidraw to svg and embed it directly in the note. This does the trick but I still feels that native support is the best. 

Which is why digital garden's [feature](https://dg-docs.ole.dev/features#excalidraw) is appealing to me because it is compatible with the excalidraw files that is embeded in the notes, and it gives a interactive scorllable canvas for the readers to view

# Where the magic fails
So far, the user experience for the entire framework is very smooth for me. The only disadvantage I found is that, pdf embeded files are not displayed. Which may or may not be a big deal for you because there are work arounds to this

# Customization
I have to say digital-garden's approach towards how users can customize their websites is really appealing to me. 
They use [Nunjucks](https://mozilla.github.io/nunjucks/templating.html) as a templating engine for their renders, you just need some simple web knowledge, you can easily customize the look of your websites.

I have done some simple tweaks to make my website look the way that I wanted

## Footnote
I want to include the `copyright` content and the `social` information in the footer of every note so that if people are interested they can contact me easily.

### How it looks
![Pasted image 20231031202405.png](/img/user/Blogs/Obisidan/attachments/Pasted%20image%2020231031202405.png)

## Banner
Banner is a very interesting idea in obisidian, notion had the same thing for their notes, not sure who's original idea was it. I feel like this can add some liveness to the full text content. This make uses of the `banner` properties defined in each note, in order for that property to be read by the code, you will have to [let all frontmatter through](https://dg-docs.ole.dev/getting-started/03-note-settings#let-all-frontmatter-through).
As the documentation mentions, it might break your site, so be careful.

### How it looks
![Pasted image 20231031202630.png](/img/user/Blogs/Obisidan/attachments/Pasted%20image%2020231031202630.png)
## Removing hash tag in TOC
In digital-garden, you can include the TOC for the current note in the side bar, but i don't really like the hashtag infront of every heading. Hence a simple css removal makes things cleaner

## What to do in the future
There are some ideas that I want to customize my website, but dont have the time to do yet.
- [ ] display the dataview table with banner and tags.
- [ ] add a link to every heading for users to copy

# Find out more
If you are interested in how this website looks ultimately, i believe you are already experiencing it. If you wish to explore more on how I did the customizations, check out the [source code](https://github.com/weihong0827/infiniteloop) 

