---
title: "Shutting Down TypeLink"
---

After 11 years I am going to be shutting down [TypeLink](https://typelink.net), the personal wiki notepad. Important logistics first, then reflections.

## Logistics

- **TypeLink.net will shut down no earlier than March 1, 2023.** After that time, no data will be accessible through the web site or iOS app.
- To export your data, sign in to the [TypeLink.net](https://typelink.net) web interface, then click “List” in the toolbar to see your list of pages, then click “Export.” You will receive a zip archive of text files. You can use these text files in any other note taking app you like. Note that the automatic links that TypeLink creates based on page titles are not preserved.
- If you have other questions, check back at this blog post—I will keep it up-to-date if any other questions come up.

There are now many notetaking apps with a lot of powerful features that TypeLink couldn’t match. Popular ones include [Bear Notes](https://bear.app/), [Obsidian](https://obsidian.md/), and [Notion](https://www.notion.so/). I was able to get my TypeLink notes loaded into Obsidian by changing the `.txt` file extensions to `.md`.

When TypeLink shuts down, I will release the source code of TypeLink's web site and iOS app in case you want to run your own instance. The link will be posted here and on [TypeLink.net](https://typelink.net). Note that the codebases are very old and I won’t be able to provide support.

## Reflections

TypeLink was my first personal application, and it represented a big step forward in my career. Previously I had only created software for other people. But with TypeLink I saw a need I had that wasn't getting met (a personal wiki notepad) and I decided to create it. I was in the Java world at the time but I had recently learned about Groovy on Grails at a conference (you won't be surprised to hear that it's a take on Ruby on Rails), and it allowed me to rapidly prototype the site. Then building the iOS app was a way to realize a lifelong dream of being an Apple platform developer. When TypeLink launched on the App Store, there were few enough apps that *every* app in a category was featured on that category's home page, and that led to a huge influx of users.

TypeLink was initially hosted on a Mac mini at my house with a Dynamic DNS IP address. I remember stressing when the server went down while I was out of town. Ultimately I learned about Digital Ocean and decided to spend the money to host it there.

In the last 11 years, I've learned that you can do side projects just for yourself; you can do them cheaply enough that you probably don't need to monetize them. You don't even need to make them publicly available! My personal apps I currently use daily are [Surely Todos](https://surelytodo.com) (which is open for free registrations) and [Firehose](https://github.com/CodingItWrong/firehose-api/blob/main/README.md), a single-user link-saving app.

I've also learned that creating apps isn't the only way to code on the side. You can do code exercises like [Exercism.org](https://exercism.org/), create throwaway repos to try things out, and write to share what you've learned.
