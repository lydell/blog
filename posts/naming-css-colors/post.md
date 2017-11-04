# Naming CSS colors

**TL;DR** If you can update the shade of a color really easily, you’ve won. Name your colors `$red`, `$blue` and `$grey-whatever`.

## The problem

Let’s say you’re implementing some design that has a lot of this blue in it. In particular, this blue: `#42aad5`. Every time you need that blue, you simply write `color: #42aad5;`, `background-color: #42aad5;`, `box-shadow: 0 0 5px #42aad5;` or what have you. So far so good.

Then one day, the designer says to you: “You know that blue we use all over the place? Can you change it to `#4bcafe`, so that it ‘pops’ a little more?”

Now you have plenty of changes to make. Easy enough, though: Just do a find-and-replace, searching for `#42aad5` and replacing with `#4bcafe`.

But shortly after your change, the designer comes to you again. “The new blue does not seem to show up on the dashboard, could you take a look at what’s going on?”

Turns out the blue had been written uppercase in the dashboard CSS for some reason: `#42AAD5`. Better remember to do a case insensitve find-and-replace next time.

But it doesn’t end there. If color-picking has been involved during the development, ever-so-slightly-off variations of the blue might exist. Perhaps your college color-picked out `#42aad6` instead of `#42aad5`. Nobody’s eyes can tell the difference, but it’s enough to fool your find-and-replace.

Or consider this shade of blue: `#44ccff`. Somebody might have chosen to write it in shorter notation: `#4cf`.

## A solution

Use a CSS preprocessor. I like [Sass]. You can also use [Less], [Stylus], [PostCSS] with some plugin, or [custom properties] in modern browsers, but I’ll go with Sass in this post.

In lots of projects I’ve worked on, there’s a `_variables.scss` file containing global variables for, well, _global_ stuff. `z-index` values, breakpoints, some font stuff–and, most importantly, all colors.

With variables for colors, the shade update would have been super easy:

```diff
-$blue: #42aad5;
+$blue: #4bcafe;
```

Teach your team to look for colors in `_variables.scss` instead of hard-coding them and you should be fine.

## The new problem

It is commonly said that one of the hardest things about programming is naming things. Which begs the question: How should you name your colors?

As a programmer, you might naturally raise your eyebrows when seeing that variable named `$blue`. Isn’t that like making doing `const TEN = 10` instead of `const VELOCITY = 10`?

You might be tempted to come up with some clever naming such as `$primary-color` and `$secondary-color`. After those two, though, I’ve basically run out of names (or at least they become harder and harder to come up with).

Besides some feeling of “purity,” what would the benefit would names like `$primary-color` provide?

Let’s say the designer changes their mind and decides green would be nicer than blue. You _could_ of course just change the `$blue` variable to the new green color instead:

```diff
-$blue: #42aad5;
+$blue: #1feb8a;
```

But then you’d leave your colleges and your future self confused.

Aha! If you’d named the variable `$primary-color` instead surely everything would have been fine?

The truth is, in my experience, that changing the _shade_ of a color is _much_ more common than changing to a whole other color. I’ve worked on a project where we switched from green and red, to blue and orange. I can tell you that the change was _not_ as easy as just swapping all red for orange, and all green for blue. Some things simply didn’t look nice then. Changing colors is basically a light-weight restyling of the entire site, so you need to go through every page anyway to see that it all looks good with the new colors. Many times, more little adjustments than just color changes are needed. `$primary-color` and `$secondary-color` would not have helped.

On the other hand, I’ve also been trough changing a pinkish color to a more purpleish. That was as easy as doing a find-and-replace searching for `$pink` and replacing with `$purple`. And we’re _not_ back at square one doing a find-and-replace again, because this time there’s no casing differences and whatnot to worry about.

Abstract color names buy you nothing other than making it harder to read and write your CSS (no matter what tools you use or how fancy your editor colors your variable names). Name your colors as simple as possible and save you some trouble. `$blue`, `$orange` and `$red` are just fine.

Simple names also make it easier to communicate with your designer. When I’m talking to designers we you usually end up saying things like “the orange for links”, “the green for buttons”, or just “the red we use.” Not “the primary color is used for links“ and “the secondary color for buttons”. We’re just simple humans after all.

So I simply choose the same names as the designer uses when talking about the design. This is especially good for colors where people disagree on the naming, such as for some shades of cyan and blue, orange and red, or beige and brown.

No more excessive color picking or pasting hex codes in chat messages! By speaking the same “language” as the designer, you give the power to simply say “use the usual project blue for that button, please,” even to a new developer.

## Yet a new problem

What about the cases where the designer does _not_ have names for some colors?

It’s easy with “colorful” colors, such as the ones I’ve been takling about (red, green, blue, orange). There’s usually just a handful of those. Having too many makes for a messy design.

The annoying one is grey. In most projects I’ve worked on, there’s always a whole slew of greys. A good designer, in my opinion, might have a large amount of different greys, but at least is consistent with using only those. If there appears to be new greys in every part of a design, it might not be worth even trying to put them in variables.

How do you name those greys? What usually happened to me was that I named the first one I came across `$grey`. Next up, `$grey-dark`. Then I might have found a lighter one: `$grey-light`. Then, `$grey-darker`. `$grey-even-darker`. `$grey-kinda-dark`. You can see where this is going.

This “sorted names scheme”It makes it hard to insert a new ones, for example between `grey-light` and `grey-lighter`. It’s also really hard to remember if it was `grey-light` or `grey-lighter` that was usually used for borders. And sometimes it’s hard to say if a grey is lighter or darker than another, and as such hard to come up with a name.

My solution is to give all the greys unique names that tell them apart, but doesn’t really say anything about the color. I’m a Star Wars fan, so I usually use Star Wars characters for the naming:

```scss
$grey-luke: #eeeef0;
$grey-lando: #969da6;
$grey-vader: #4d4d55;
```

You your imagination! Choose something that you like–and your colleges know about. If you’re the only Star Wars fan around, choosing another topic might be wiser.

If you want to take it a step further, you can let the name of a grey color give a _hint_ about how dark it is. `$grey-luke` is probably a light grey, while `$grey-vader` is probably darker. Star Wars is good in this aspect, since the characters are usually obviously good (light) or evil (dark).

I think it’s fine to have `$green-light` and `$green-dark`. It’s when there’s three or more shades that things become hard. I’ve worked on a project that had almost as many shades of green as grey, and the Star Wars naming worked out there too.

It felt strange at first seeing `border: 1px solid $grey-luke;` in my CSS, but after a little while you almost grow a relationship with these “characters.” And I find it easier to tell `$grey-luke` and `$grey-leia` apart than `$grey-light` and `$grey-lighter`.

## How far should you go?

Should _every_ color be defined in `_variables.scss`? Only as long as it’s practical.

I usually use the regular old `white` and `black` CSS keywords for absolute whites and blacks. I’ve never had an issue with that so far.

Sometimes there are special parts of designs with one-off colors. I usually put them inline in the relevant .scss file and leave a comment that they are one-off. If the same one-off color is used several times within that .scss file, remember that you can make a local variable just for that file.

## Conclusion

Colors are important. Make them easy to use. Easy to change. Easy to talk about with other people. Don’t make naming hard for yourself. Be practical. And allow a little fun in your CSS :)
