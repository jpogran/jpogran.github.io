---
title: Setting Up Things
category:
    - blog
tags:
    - general
---

I have been wanting to setup a blog again for awhile, but also still maintain some level of control over both the nuts and bolts as well as an easy method to author posts. I have been reading about GitHub's jekyll for awhile now, and found a project that gives you a pretty nice basic site template called poole.

Following a nice guide http://joshualande.com/jekyll-github-pages-poole, I was able to get a site up and running on GitHub pretty easily. In fact, the hardest part figuring out how to get Github resolving my domain name.

- Create githhub repo username.github.io
- You can use the auto site generator feature, or follow the instructions to manually create one. If you chose the auto site generator then you have to delete the files in the next step
- If you chose the auto site generator, you now have a basic site already
    - Check it out at http://username.github.io
- Pull down your repository using git
    - `git clone https://github.com/username/username.github.io`
- Delete the existing content if you chose the auto site genertor
- Copy the poole site files into your repo
- Add a file called `CNAME`. *Note* case is important, no file extension
- Add your custom domain to the `CNAME` file
- Push your changes
