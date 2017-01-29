---
title: Migrating from WordPress.com to GitHub Pages
categories: miscellaneous
---
This blog was [originally hosted](https://toddtaomae.wordpress.com) on
[WordPress.com](https://wordpress.com) but I've decided to migrate it to
[GitHub Pages](https://pages.github.com) for various reasons. This post will outline my reasons for
migrating as well as a comparison of my experiences with the two services.

For those who aren't familiar with WordPress.com or GitHub Pages, they are both web hosting
services. WordPress.com is powered by [WordPress](https://wordpress.org/). It can be a little
confusing, but basically WordPress.com hosts sites powered by WordPress. Similarly, GitHub Pages
hosts sites powered by [Jekyll](https://jekyllrb.com/). You could also use either WordPress or
Jekyll to host a site on your own or through some other web hosting service. In this post I will
sometimes simply say "WordPress" or "GitHub Pages" and depending on the context, that may refer to
the web hosting service (WordPress.com and GitHub Pages), the underlying software (WordPress and
Jekyll), or a combination of both.

# Motivation for Migrating
My primary motivation for migrating to GitHub Pages was so that everything related to my résumé
and professional portfolio would be hosted on one site. This is mostly a convenience for myself,
since that means there is one less account that I have to manage, but I also like the idea of
everything being a bit more "unified." Another major motivating factor was simply to experiment with
some new (to me) technology ([Jekyll](https://jekyllrb.com/), [Sass](http://sass-lang.com/), and
[Liquid](http://shopify.github.io/liquid/)) and gain some experience building a simple, static
website.

There were also several other, minor factors that contributed towards my decision. First is the
ability to have more control over the content and layout of the site. That's not to say that
WordPress isn't customizable, but there were a few minor annoyances that I'm glad I won't have to
deal with anymore. In particular, I didn't like how file uploads were handled. Files are
automatically placed in a _[year]/[month]_ directory, which makes sense in many cases, such as when
adding an image to a post. However, in other cases, such as my résumé, I don't want the URL to
change every time I upload a new version. Another related issue, is if I want to update a file in
the same month, I believe it would create a new file with an added suffix and keep the old file,
even if I intended to completely replace the old file[^disclaimer1].

The last reason that I chose to migrate was so that everything would be under version control.
In most cases, there is probably not much benefit since most content won't change much and new posts
which is the bulk of the content, are given dates. However, in the cases where I do edit posts or
other content, it can be nice to have the version history. Having the site be a Git repository also
means that if something were to ever happen to GitHub, I can keep a local copy and deploy the site
somewhere else.

# WordPress Versus GitHub Page
For my purposes both WordPress and GitHub Pages have all the features I need to produce the content
that I want, so my comparison will focus on ease of use, customizability, and extra features.

In general, WordPress.com is easier to use than GitHub Pages since it has a web interface and
dashboard to handle almost everything. You never need to look at or even understand HTML or CSS to
be able to create a site. The same is true to a certain extent for GitHub Pages, but if you want
anything beyond what the available themes provide by default, you will need to have an understanding
of how Jekyll works as well as some understanding of HTML and/or CSS. However, the fact that you can
directly control the HTML and CSS that is output by Jekyll and ultimately served to the user means
that there are more options for customization[^disclaimer2].

One thing that I liked about Jekyll was the simplicity of intalling and running a local server. This
means that you can easily make changes locally and see exactly what it will look like when it is
deployed to GitHub Pages and it is easy to view on different devices and browsers. While it is
possible to run WordPress locally, it appears to be significantly more involved, although I have
never actually tried it so my opinion is mostly based on viewing the documentation.

One major feature missing from GitHub Pages that was available with WordPress.com was web analytics,
which provided information such as the number of unique visitors, what country visitors are from,
and the number of views per page. I've decided to use
[Google Analytics](https://www.google.com/analytics/) as an alternative. In some ways, this opposes
the fact that I wanted everything to be hosted on one site. However, since it is tied to my Google
account, which I already use daily for email, I decided it was worth the tradeoff.

Overall, I am happy with my decision to switch to GitHub Pages. Although it came at the cost of some
ease of use, the additional experience that I gained with HTML, CSS, and, to a certain extent, user
interfaces and usability, was worth it.


[^disclaimer1]: **Disclaimer:** It's been quite a while since I've dealt with this so I may be
    mis-remembering some of the details. Things may have also changed since then or I may have even
    just missed something when I originally ran into the problem.

[^disclaimer2]: **Disclaimer:** I don't know a lot about WordPress and I haven't really spent much
    time exploring configuration and customization through WordPress.com so I may be completely
    wrong about the amount of customization that is available.
