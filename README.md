# TypeScript + Node.js + ESLint + Prettier + Cool Stuff

TypeScript project from scratch with cold-reloading, and scripts for building, development, and production environments.

## ts config setup

```
npx tsc --init --rootDir src --outDir build \
--esModuleInterop --resolveJsonModule --lib es6 \
--module commonjs --allowJs true --noImplicitAny true
```

- `rootDir`: This is where TypeScript looks for our code. We've configured it to look in the src/ folder. That's where we'll write our TypeScript.
- `outDir`: Where TypeScript puts our compiled code. We want it to go to a build/ folder.
- `esModuleInterop`: If you were in the JavaScript space over the past couple of years, you might have recognized that modules systems had gotten a little bit out of control (AMD, SystemJS, ES Modules, etc). For a topic that requires a much longer discussion, if we're using commonjs as our module system (for Node apps, you should be), then we need this to be set to true.
- `resolveJsonModule`: If we use JSON in this project, this option allows TypeScript to use it.
- `lib`: This option adds ambient types to our project, allowing us to rely on features from different Ecmascript versions, testing libraries, and even the browser DOM api. We'd like to utilize some es6 language features. This all gets compiled down to es5.
- `module`: commonjs is the standard Node module system in 2019. Let's use that.
- `allowJs`: If you're converting an old JavaScript project to TypeScript, this option will allow you to include .js files among .ts ones.
- `noImplicitAny`: In TypeScript files, don't allow a type to be unexplicitly specified. Every type needs to either have a specific type or be explicitly declared any. No implicit anys.

## Scripts overview

```
npm run start:dev
```

Starts the application in development using nodemon and ts-node to do cold reloading.

```
npm run build
```

Builds the app at build, cleaning the folder first.

```
npm run start
```

Starts the app in production by first building the project with npm run build, and then executing the compiled JavaScript at build/index.js.

## Dev Tools

- `rimraf`: a cross-platform tool that acts like the rm -rf command (just obliterates whatever you tell it to).
- `ts-node`: for running TypeScript code directly without having to wait for it be compiled
- `nodemon`: to watch for changes to our code and automatically restart when a file is changed
- `@types/node`: TypeScript has Implicit, Explicit, and Ambient types. Ambient types are types that get added to the global execution scope. Since we're using Node, it would be good if we could get type safety and auto-completion on the Node apis like `file`, `path`, `process`, etc. That's what installing the [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) type definition for Node will do.

# ESLint and TSLint

ESLint is a JavaScript linter that enables you to enforce a set of style, formatting, and coding standards for your codebase. It looks at your code, and tells you when you're not following the standard that you set in place.

You may have also heard of TSLint, the TypeScript equivalent. In 2019, the team behind TSLint decided that they would no longer support it. The reason, primarily, is because ESLint exists, and there was a lot of duplicate code between projects with the same intended purpose.

## Installation and setup

```
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

## Basic Starter Config for .eslintrc file

```
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint"
  ],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended"
  ]
}
```

## Ignoring files we don't want to lint with `.eslintignore` file

```
node_modules
build
```

## Adding a lint script + Fixing linted code

```
{
  "scripts": {
    ...
    "lint": "eslint . --ext .ts",
    "lint-and-fix": "eslint . --ext .ts --fix"
  }
}
```

## ESLint Rules

There are three modes for a rule in eslint: off, warn, and error.

- `off`: means 0 (turns the rule off completely)
- `warn`: means 1 (turns the rule on but won't make the linter fail)
- `error`: means 2 (turns the rule on and will make the linter fail)

## Adding a rule

In .eslintrc, add a new attribute to the json object called "rules".

Rules get added as keys of this rules attribute, and you can normally find the eslint base rules here on their website via the Rules [docs](https://eslint.org/docs/rules/).

Example: We want to add the no-console rule, so here is an example of how we can make the linter fail (throw a mean error code) if it encounters a console.log statement with the no-console rule set to error.

```
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  ...
  "rules": {
    "no-console": 2 // Remember, this means error!
  }
}
```

## Using rules in a real-life project

There's a reason that all three of these modes exist. When you set the linter to off for a rule, it's because you and your team probably don't care about that rule in a particular base configuration you're using (we're currently using the ESLint recommended config but you can also go and use [Shopify's](https://github.com/Shopify/eslint-plugin-shopify), [Facebook's](https://www.npmjs.com/package/eslint-config-fbjs) or several other companies' configs as starting points instead).

When you set the rule to `error - 2`, it means that you don't want the code that breaks your coding conventions to even make it into the repo at all.

To remedy overly restrictive rules, the `warn - 1` setting means that yes, you want you and your team to adhere to that rule, but you don't want it to prevent you from moving forward.

## Adding a plugin (features)

ESLint also allows you to add one-off features to your config. These are known as plugins.

Here's a fun one. It's called [no-loops](https://github.com/buildo/eslint-plugin-no-loops).

> Check out this list of other [awesome-eslint](https://github.com/dustinspecker/awesome-eslint) plugins and configs.

```
npm install --save-dev eslint-plugin-no-loops
```

And then update your `.eslintrc` with `no-loops` in the "plugins" array, and add the rule to the "rules" attribute like so.

```
{
  "root": true,
  ...
  "plugins": [
    "@typescript-eslint",
    "no-loops"
  ],
  ...
  "rules": {
    "no-console": 1,
    "no-loops/no-loops": 2
  }
}
```

Now if we update our code to include a for loop, And run the lint command again...We'll see the errors that restricts loops appear.

## Extending a different base config

Start with a different base config- Shopify's, for example.

Looking at their [readme](https://github.com/Shopify/eslint-plugin-shopify), we need to install it by running:

```
npm install eslint-plugin-shopify --save-dev
```

Update `.eslintrc`

```
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  ...
  "extends": [
    "plugin:shopify/esnext" // this
  ],
  ...
}
```

> You can add several base configs to your project by including them in the array, though you may end up seeeing the same linting rules twice or more.

# Prettier with ESLint

Prettier with ESLint, delegating the responsibility of code convention definition to ESLint, and the responsibility of formatting to Prettier.

## Using Prettier with ESLint

Combining both ESLint and Prettier, the responsibility division is this:

- ESLint defines the code conventions
- Prettier performs the auto-formatting based on the ESLint rules

## Installing Prettier as dev dep

```
npm install --save-dev prettier
```

## Configuring Prettier

As per the [docs](https://prettier.io/docs/en/configuration.html), we can expose a JSON file called `.prettierrc` to put our configuration within.

```
touch .prettierrc
```

A basic `.prettierrc` setting is the following:

```
{
  "semi": true,
  "trailingComma": "none",
  "singleQuote": true,
  "printWidth": 80
```

}

These settings specify the following rules:

- `semi`: set to `true` means that Prettier will add semicolons when necessary.
- `trailingComma`: set to `none` means that Prettier will remove any trailing commas at the end of objects.
- `singleQuote`: set to `true` means that Prettier will automatically use single quotes instead of double quotes.
- `printWidth`: set to `80` specifies that the printer will wrap any lines that exceed 80 characters.

You can view the rest of the options [here](https://prettier.io/docs/en/options.html) and change them as you like! This is just my personal preference.

## Script to execute prettier from package.json

```
{
  "scripts": {
    ...
    "prettier-format": "prettier --config .prettierrc 'src/`/*.ts' --write"
  }
}
```

## Configuring Prettier to work with ESLint (Here's where the real magic happens)

With ESLint and Prettier already installed, install these two packages as well.

```
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

- `eslint-config-prettier`: Turns off all ESLint rules that have the potential to interfere with
  Prettier rules.
- `eslint-plugin-prettier`: Turns Prettier rules into ESLint rules.

Lastly, we need to make an adjustment to the `.eslintrc`.

```
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "plugins": [
    ...,
    "prettier" // here
  ],
  "extends": [
    ...,
    "prettier" // here
  ],
  "rules": {
    ...,
    "prettier/prettier": 2 // Means error // here
  }
}
```

If you're not interested in using the recommended plugins, the most basic `.eslintrc` with Prettier enabled looks like this:

```
{
  "extends": ["prettier"],
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": 2 // Means error
  }
}
```

Now Running `npm run lint` lints the files and tells us all of the unformatted lines in addition to any code convention violations we've protected against through ESLint rules.

That's it! Run `npm run prettier-format` to format everything.

## Formatting using VSCode on save (recommended)

Install the [Prettier VS Code extension](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode).

To set the defaults, press `CMD + SHIFT + P` (on MacOS) or `CTRL + Shift + P` (on Windows), then type in `preferences open settings`.

You want to select the JSON option so that we can manually edit the preferences via a JSON file.

In the JSON file, if it's not already added, add the following lines to the root of the object.

```
{
    // Default (format when you paste)
    "editor.formatOnPaste": true,
    // Default (format when you save)
    "editor.formatOnSave": true,
    // Disable for specific language
    "[javascript]": {
        "editor.formatOnPaste": false,
        "editor.formatOnSave": false,
    },
}
```

> These settings will format your code both when you paste new code and when you save code for any file extension that Prettier understands. If the root of the project that the file lives in has a `.prettierrc`, it will use the settings that live in that file to format your code.

## Formatting before a commit - Enforcing Coding Conventions with Husky Pre-commit Hooks (recommended)

When working with other developers as a team, it can be challenging to keep the formatting of the code clean constantly. Not everyone will want to use the Prettier VS Code extension. Not everyone will want to use VS Code!

How do we ensure that any code that gets commited and pushed to source control is properly formatted first?

Husky to prevent bad git commits and enforce code standards in your project.

When you initialize Git (the version control tool that you're probably familar with) on a project, it automatically comes with a feature called hooks.

If you go to the root of a project intialized with Git and type:

```
ls .git/hooks
```

You'll see a list of sample hooks like `pre-push`, `pre-rebase`, `pre-commit`, and so on. This is a way for us to write plugin code to execute some logic before we perform the action.

If we wanted to ensure before someone creates a commit using the git commit command, that their code was properly linted and formatted, we could write a `pre-commit` Git hook.

## Installing Husky

To install Husky, run:

```
npm install husky --save-dev
```

## Configuring Husky

To configure Husky, in the root of our project's `package.json`, add the following `husky` key:

```
"husky": {
  "hooks": {
    "pre-commit": "", // Command goes here
    "pre-push": "", // Command goes here
    "...": "..."
  }
}
```

When we execute the git commit or git push command, the respective hook will run the script we supply in our package.json.

## Example workflow

Following along from the previous configured ESLint and Prettier, let's utilize two scripts:

- `prettier-format`: Format as much code as possible.
- `lint`: Ensure that the coding conventions are being adhered to. Throw an error if important conventions are broken.

`package.json`

```
{
  "scripts": {
    "prettier-format": "prettier --config .prettierrc 'src/**/*.ts' --write",
    "lint": "eslint . --ext .ts",
    ...
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run prettier-format && npm run lint"
    }
  }
}
```

Include these `scripts` in the scripts object in the `package.json`. Then, at the very least, run `prettier-format` and then `lint` as a `pre-commit` hook.

This will ensure that you cannot complete a `commit` without formatted code that passes the conventions.

## Other considerations

If you notice that linting is taking a long time, check out this package, [lint-staged](https://github.com/okonet/lint-staged). It runs the linter, but only against files that are staged (files that you're ready to push).

## Formatting using an filesystem watcher

If you really don't like VS Code or there are too many people on your team not using it, there's another option to tighten the feedback loop of formatting code as you're writing it.

The [Prettier docs](https://prettier.io/docs/en/watching-files.html) suggest using a package called `onchange` in order to watch the filesystem for when changes are made to your source code, then run the Prettier CLI tool against any changed files.

Here's how that works.

Install `onchange`.

```
npm install - -save-dev onchange
```

Then, add this script to your `package.json`, making sure to change the watched directory if you keep your source code in a location different the `src` folder.

```
"scripts": {
    "prettier-watch": "onchange 'src/\*_/_.ts' -- prettier --write {{changed}}"
}
```

Opening a new console, running `prettier-watch`, and editing code within the `src` folder will give us the same observable outcome as if we set Prettier up with VS Code or not.

## Manually formatting a single file

On MacOS, if you've installed the VS Code extension, you can format the current file by typing `SHIFT + OPTION + F`.

This might be different for you. You can see what the command is by typing `COMMAND + SHIFT + P` and entering "format".

---

Whole setup is based on these articles (all credits goes to [@khalilstemmler](https://twitter.com/stemmlerjs)):

- https://khalilstemmler.com/blogs/typescript/node-starter-project/
- https://khalilstemmler.com/blogs/typescript/eslint-for-typescript/
- https://khalilstemmler.com/blogs/tooling/prettier/
- https://khalilstemmler.com/blogs/tooling/enforcing-husky-precommit-hooks/
