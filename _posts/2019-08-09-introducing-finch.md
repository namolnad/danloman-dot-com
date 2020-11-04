---
layout: post
title:  "Introducing Finch 🐦 — A conventional-commit powered changelog generator"
date:   2019-08-09
tags: swift tooling changelog
---

Hello, World! Meet [Finch][repo], a configurable commandline tool, built in Swift, designed to easily create and format changelogs. Finch, meet the World. 🤝

---

A good changelog can save a lot of potential headaches down the road. Whether it be to report to stakeholders when a change was introduced, to let customers know what new features are coming out, or to aid developers who are tracking down a new bug, it can be invaluable to keep track of when every feature, bug fix, or otherwise substantial change came about.

Unfortunately, developers already have more than enough on their plates - which often leaves changelogs untimely, completely forgotten, or in the best case, difficult to maintain stylistically between various team members. These problems are what inspired the creation of [Finch][repo].

![Finch git log](/assets/finch-git-log.png)
*Viewing Finch's raw git log which powers the generated changelog*

Through the use of well-formed and intentional Git commit messages, Finch can very easily convert your Git commits into a consistently formatted, and fully automated changelog. The commit messages serve as the underlying data which is then passed through a formatting system according your project's custom configuration. The only requirement Finch has is the use of some relatively minor commit-message discipline - the simple use of "tag" prefixed commit messages. For example:

> git commit -m '[feature] Add the bells. And the whistles'

With just that — and according to whatever conventions your team would like to use - Finch can help you automate your internal and external-facing changelogs, providing as much detail or polish as is desired. All you have to do is run `finch compare` and Finch will take care of the rest.

![Finch example command](/assets/finch-example-command.png)
*Using Finch to generate a changelog between two recent versions of Finch. Super meta.*

It's that simple! And the above raw markdown renders beatifully as your formatted changelog:

![Finch exmaple output](/assets/finch-example-output.png)
*The Finch changelog as fully rendered markdown - so simple!*

Check it out [here][repo] and let me know what you think!

[repo]: https://github.com/namolnad/safeguard