# nyum

*A simple Pandoc-powered static site generator for your recipe collection.*

<img src="_assets/favicon.png" align="right" width="96">

This tool takes a **collection of Markdown-formatted recipes** and turns it into a **lightweight, responsive, searchable website** for your personal reference. It's not intended for use as a cooking blog – there's no RSS feed, no social sharing buttons, and absolutely zero SEO.

📓 Think of it as a **cookbook for nerds**.

Below the screenshots, you'll find some notes on [setting this tool up](#setup), [running it](#building), [formatting your recipes](formatting), and [deploying the generated site](#deployment).


## Screenshots

If you prefer a live website over the following screenshots, feel free to **check out the [demo on GitHub Pages](https://doersino.github.io/nyum/_site/index.html)**!

On an old-fashioned computer, a [recipe](https://doersino.github.io/nyum/_site/cheesebuldak.html) might look more or less like this – notice the little star indicating that this is a favorite!

![](_assets/readme-images/1-laptop.jpg)

Below, on the right, is the same page shown at tablet scale. More interestingly, the index page is shown on the left (with an active search) – note that you can, of course, customize the title and description.

![](_assets/readme-images/2-tablets.jpg)

Finally, more of the same on three phone-sized screens. The three-column layout doesn't fit here, so instructions are shown below ingredients.

![](_assets/readme-images/3-phones.jpg)


## Usage

### Setup

First off, either `git clone` this repository or [download it as a ZIP](https://github.com/doersino/nyum/archive/refs/heads/main.zip). (You can clear out the `_recipes/` and `_site/` directories to get rid of the demo data.)

I don't like complicated dependency trees and poorly-documented build processes, so here's an **exhaustive list of the dependencies** you're not overwhelmingly likely to already have:

* [Pandoc](https://pandoc.org) – any version released in the last few years will probably work.

    On macOS, assuming you're using [Homebrew](https://brew.sh), `brew install pandoc` will do the trick. On Linux, your package manager almost certainly has it. As for Windows: I haven't used it in a decade, so you're on your own.

That's it, only one dependency! Hooray!

(Since `build.sh` relies on some Bash-specific bits and bobs, you'll also need that – but since it's the default shell on most systems, you're likely running it already.)


### Configuration

Open `config.yaml` in whichever text editor you heart is drawn to in the moment and follow the instructions in the comments. There's not actually very much to configure.


### Building

Run `bash build.sh`.

(It accepts a few optional flags, notably `--help` which tells you about the rest of them.)

![](_assets/readme-images/4-build.gif)


### Formatting

TL;DR: See the example recipes in `_recipes/`.

Each recipe **begins with YAML front matter specifying its title**, how many servings it produces, whether it's spicy or vegan or a favorite, the category, an image (which must also be located in the `_recipes/` directory), and other information. Most of these are optional!

The **body of a recipe consists of horizontal-rule-separated steps, each listing ingredients relevant in that step along with the associated instruction**. Ingredients are specified as an unordered list, with ingredient amounts enclosed in backticks (this enables the columns on the resulting website). The instructions must be preceded with a `>`. Note that a step can also solely consist of an instruction.

*You've got the full power of Markdown at your disposal – douse your recipes in formatting, include a picture for each step, and use the garlic emoji as liberally as you should be using garlic in your cooking!*

```markdown
---
title: Cheese Buldak
original_title: 치즈불닭
category: Korean Food
description: Super-spicy chicken tempered with loads of cheese and fresh spring onions. Serve with rice and a light salad – or, better yet, an assortment of side dishes.
image: cheesebuldak.jpg
size: 2-3 servings
time: 1 hour
author: Maangchi
source: https://www.youtube.com/watch?v=T9uI1-6Ac6A
spicy: ✓
favorite: ✓
---

* `2 tbsp` chili flakes (gochugaru)
* `1 tbsp` gochujang
* `½-⅔ tbsp` soy sauce
* `1 tbsp` cooking oil
* `¼ tsp` pepper
* `2-3 tbsp` rice or corn syrup
* `2 tbsp` water

> Mix in an oven-proof saucepan or cast-iron skillet – you should end up with a thick marinade.

---

* `3-4 cloves` garlic
* `2 tsp` ginger

> Peel, squish with the side of your knife, then finely mince and add to the marinade.

---

> ⋯ (omitted for brevity)

---

> Garnish with the spring onion slices and serve.

```

(Before building this tool, I was using a custom LaTeX template built on top of the [cuisine](https://ctan.org/pkg/cuisine?lang=en) package, which enforces a three-column, relevant-ingredients-next-to-instructions structure. [In the process of graduating from university, I found myself contemporaneously graduating from wanting to use LaTeX for everything, which was part of the impetus for building this tool.] I've found this structure to be more useful than the more commonly found all-ingredients-first-then-a-block-of-instructions approach.)


### Deployment

After running `build.sh`, **just chuck the contents of `_site/` onto a server of your choice**.

For my own convenience, I've written `deploy.sh`, which reads a remote target of the form `USER@SERVER:PATH` from `config.yaml` and then uses `rsync` to push `_site/` cloudwards – you're welcome to use it, too. If you do:

* Note that `rsync`'s `--delete` flag is set, so make sure the target path is correct before deploying for the first time. If you don't, stuff that shouldn't be deleted or overwritten might indeed be deleted or overwritten!
* You'll need to manually create the target path on the remote before the first deployment.
* You can run `bash deploy.sh --dry-run` to sanity-check yourself.
* Run `bash deploy.sh --help` to learn about another very exciting flag!


## FAQ

(As in "𝓕ound, by me, to be likely-to-be-𝓐sked 𝓠uestions, the reason being that I asked these questions to myself during construction of this thing".)


### Why not just use Jekyll or one of the other 314 fully-featured static site generators out there?

Because I thought writing a Bash script where I construct a JSON value based on other JSON values using a single-purpose reimplementation of SQL's `GROUP BY` clause reliant on the built-in string manipulation functionality would be simpler/faster/better, *i.e.*, **because I'm a dummy**.

(But, newly, a dummy armed with a custom dodgy-yet-working static site generator, so you better not cross me!)


### How/why does that huge mess in `build.sh` work?

Apart from the translation of Markdown into HTML, which is a fairly self-explanatory `pandoc` call, and the `config.yaml` shenanigans, which are merely a medium-sized mess: I wanted to **build an index page listing all recipes, but ordered by category** and with cute spicy/vegan/etc. icons.

Each recipe has a set of metadata (specified using YAML, but that's not relevant here), including its category. When outputting HTML, Pandoc provides the `$meta-json$` template variable which resolves to a JSON value containing this metadata. Crucially, it understands the same format during input – when invoking `pandoc` with the `--metadata-file PATH` flag, the metadata from that file is merged into the input's metadata before further processing. The challenge, then, was **transforming the JSON-shaped metadata of all recipes into a single JSON value grouping them by category**.

This led me down the path of...

1. Writing the metadata of each recipe to a JSON file in `_temp/` by feeding them into Pandoc and using a template solely consisting of `$meta-json$`.
2. Writing the paths of each metadata files, along with the associated category, to a separate file in `temp/` using a similar minimal template.
3. Employing a `cut`-`sort`-`uniq` pipeline to distill a list of unique categories.
4. Using a good ol' bespoke nested-loops join for grouping, *i.e.*, iterating through the list of categories and for each category, writing its name to the output JSON file before iterating though the list of paths-and-categories from step 2 to figure-out-the-path-of-and-collect the recipe metadata belonging to the current category.

The final implementation is a bit more complicated than this pseudocode – largely because I, for some reason and despite knowing better, decided to "gracefully" deal with uncategorized recipes.

Building the search "index" works similarly, but without the need for any grouping shenanigans.


### Since this static site generator is based around a Bash script and Bash is a terrible language as far as robust string manipulation is concerned, are there any pitfalls with regard to filenames and such?

Why, there are indeed! I'm 100% sure these could be remedied quite easily, but they don't interfere with my use case, so I didn't bother. If you run into any problems because of this, please [file an issue](https://github.com/doersino/nyum/issues) or cancel me on Twitter.

* No spaces in filenames.
* No section signs `§` in category names.
* Things will probably break if `_recipes/` is empty (but then, there's not much to be done in that case, anyway).
* You can't have a recipe with filename `index.md` – it'll be overwritten by the generated index page.
* The value of `uncategorized_label` in `config.yaml` may not contain an odd number of double quotation marks `"`.
* *Almost certainly more!*


### What if I want to print one of the recipes with black water on dead wood?

While this isn't a use case I'm particularly interested in, I've added a few CSS rules that should help with it.


### How's browser support looking?

The CSS I've written to render Pandoc's output in three columns is a bit fragile, but I've successfully coaxed it into yielding near-identical results in recent versions of Firefox, Chrome and Safari. If you run into any problems, please [file an issue](https://github.com/doersino/nyum/issues).


### Any plans for future development?

Eh, not really. Maybe a dark mode. Or more powerful search, *e.g.*, being able to search based on labels, or navigating search results with the arrow keys. And *content*, but that won't be publicly available.


### Is there a C-based tool that's much better but not yours, so your not-invented-here syndrome didn't permit you to use it?

I think you might be alluding to Hundred Rabbits' [Grimgrains](https://github.com/hundredrabbits/GrimGrains). Big fan.


### What's the dish in the background of `_assets/favicon.png`?

That's the supremely tasty and [even more Instagram-worthy](https://www.instagram.com/p/B6vQOHDiySF/) "Half-Half Curry" served at [Monami Curry, Yongsan-gu, Seoul](https://www.google.com/maps/place/Monami+Curry+Seoul/@37.5298686,126.9707568,15z/data=!4m5!3m4!1s0x0:0x6ce40a80f13a74d5!8m2!3d37.5298686!4d126.9707568).


## License

You may use this repository's contents under the terms of the *MIT License*, see `LICENSE`.

However, the subdirectories `_assets/fonts/` and `_assets/tabler-icons` contain **third-party software with its own licenses**:

* The sans-serif typeface [**Barlow**](https://github.com/jpt/barlow) is licensed under the *SIL Open Font License Version 1.1*, see `_assets/fonts/barlow/OFL.txt`.
* [**Lora**](https://github.com/cyrealtype/Lora-Cyrillic), the serif typeface used in places where Barlow isn't, is also licensed under the *SIL Open Font License Version 1.1*, see `_assets/fonts/lora/OFL.txt`.
* The icons (despite having been modified slightly) are part of [**Tabler Icons**](https://tabler-icons.io), they're licensed under the *MIT License*, see `_assets/tabler-icons/LICENSE.txt`.

Finally, some **shoutouts** that aren't *really* licensing-related, but fit better here than anywhere else in this document:

* The device mockups used to spice up the screenshots in this document are from [Facebook Design](https://design.facebook.com/toolsandresources/devices/).
* Because you're dying to know this, let me tell you that the screenshots' background image is based on a [Google Maps screenshot of a lithium mining operation in China](https://twitter.com/doersino/status/1324367617763676160).
* I've designed the logo using a previous project of mine, the [Markdeep Diagram Drafting Board](https://doersino.github.io/markdeep-diagram-drafting-board/).
