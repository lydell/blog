# How to create a package.json – the Right Way™

Lots of guides on the Internet simply tell you to run `npm init`. But really, first you should be asking yourself a question:

**Are you going to publish your project on the [npm] registry?**

- Yes: `npm init`
- No: `echo '{"private": true}' > package.json`

## Why

package.json files are used for two things. They define the name, version, description and so on for packages that can be installed from the [npm] registry. They are also used for various configuration in all sorts of JavaScript projects.

Let's start with npm packages. In this case, the `name` and `version` fields are required to be able to publish:

```console
$ echo '{}' > package.json
$ npm publish
npm ERR! package.json requires a "name" field

$ echo '{"name": "mywebsite.com"}' > package.json
npm ERR! package.json requires a valid "version" field

$ echo '{"name": "mywebsite.com", "version": "1.2.3"}' > package.json
$ npm publish
+ mywebsite.com@1.2.3
# (Ok, I didn't actually publish that for this demo.)
```

Running `npm init` will prompt you for `name` and `version` as well as other useful fields, making it easy to get started with your new npm package. Not much to say there. The interesting case is when you _aren't_ making an npm package.

Let's say you're making a website and use npm to install jQuery.

If you start out using `npm init` you'll end up with a package.json valid for publication. So when you are multi-tasking and accidentally run `npm publish` in the wrong terminal window, you have suddenly made your entire website source code available publicly on the Internet.

`"private": true` to the rescue! Adding this to package.json will make npm refuse to publish:

```console
$ echo '{"name": "mywebsite.com", "version": "1.2.3", "private": true}' > package.json
$ npm publish
npm ERR! This package has been marked as private
npm ERR! Remove the 'private' field from the package.json to publish it.
```

Most likely you have absolutely no need to specify `name` and `version` (as well as `description`, `author` and `license`) in package.json for a website, so you could just as well remove them.

If we remove `name` and `version`, doesn't that make `private` redundant as well? As seen above, npm will refuse to publish if those fields are missing anyway. The answer is: No. Here's why:

```console
echo '{}' > package.json
$ npm install jquery
npm WARN package No description
npm WARN package No repository field.
npm WARN package No license field.

+ jquery@3.2.1
added 1 package in 0.806s
```

See those warnings? npm is going to nag you about missing fields every time you install something.

I guess it's a good thing that npm bugs package developers about this frequently, since those fields are very useful. But not for a website. Luckily, npm understands this and doesn't nag you if you tell it that you won't publish your project. Like adding that `private` field.

```console
$ echo '{"private": true}' > package.json
$ npm install jquery
+ jquery@3.2.1
added 1 package in 0.805s
```

No more warnings!

## Conclusion

For npm packages, `npm init` is a great way to guide you through the needed fields.

In all other cases, `"private": true` is all you need upfront. No unused cruft in package.json, and no risk of accidental `npm publish`.

Note: This was tested with npm 5.3.0.

[npm]: https://www.npmjs.com/
