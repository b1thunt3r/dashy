# Development Guides

A series of short tutorials, to guide you through the most common development tasks. 

Sections:
- [Creating a new theme](#creating-a-new-theme)
- [Writing Translations](#writing-translations)
- [Adding a new option in the config file](#adding-a-new-option-in-the-config-file)
- [Updating Dependencies](#updating-dependencies)
- [Writing Netlify Cloud Functions](#developing-netlify-cloud-functions)
- [Hiding Page Furniture](#hiding-page-furniture-on-certain-routes)
- [Adding / Using Environmental Variables](#adding--using-environmental-variables)
- [Building a Widget](#building-a-widget)

## Creating a new theme

Adding a new theme is really easy.  There's two things you need to do: Pass the theme name to Dashy, so that it can be added to the theme selector dropdown menu, and then write some styles!

##### 1. Add Theme Name
Choose a snappy name for you're theme, and add it to the `builtInThemes` array inside [`defaults.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/defaults.js#L27). 

##### 2. Write some Styles!
Put your theme's styles inside [`color-themes.scss`](https://github.com/Lissy93/dashy/blob/master/src/styles/color-themes.scss). 
Create a new block, and make sure that `data-theme` matches the theme name you chose above. For example:

```css
html[data-theme='tiger'] {
  --primary: #f58233;
  --background: #0b1021;
}
```

Then you can go ahead and write you're own custom CSS. Although all CSS is supported here, the best way to define you're theme is by setting the CSS variables. You can find a [list of all CSS variables, here](https://github.com/Lissy93/dashy/blob/master/docs/theming.md#css-variables).

For a full guide on styling, see [Theming Docs](./theming.md).

Note that if you're theme is just for yourself, and you're not submitting a PR, then you can instead just pass it under `appConfig.cssThemes` inside your config file. And then put your theme in your own stylesheet, and pass it into the Docker container - [see how](https://github.com/Lissy93/dashy/blob/master/docs/theming.md#adding-your-own-theme).

## Writing Translations

For full docs about Dashy's multi-language support, see [Multi-Language Support](./multi-language-support.md)

Dashy is using [vue-i18n](https://vue-i18n.intlify.dev/guide/) to manage multi-language support.

Adding a new language is pretty straightforward, with just three steps:

##### 1. Create a new Language File
Create a new JSON file in `./src/assets/locales` name is a 2-digit [ISO-639 code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for your language, E.g. for German `de.json`, French `fr.json` or Spanish `es.json` - You can find a list of all ISO codes at [iso.org](https://www.iso.org/obp/ui).

##### 2. Translate!
Using [`en.json`](https://github.com/Lissy93/dashy/tree/master/src/assets/locales/en.json) as an example, translate the JSON values to your language, while leaving the keys as they are. It's fine to leave out certain items, as if they're missing they will fall-back to English. If you see any attribute which include curly braces (`{xxx}`), then leave the inner value of these braces as is, as this is for variables.

```json
{
  "theme-maker": {
    "export-button": "Benutzerdefinierte Variablen exportieren",
    "reset-button": "Stile zurücksetzen für",
    "show-all-button": "Alle Variablen anzeigen",
    "save-button": "Speichern",
    "cancel-button": "Abbrechen",
    "saved-toast": "{theme} Erfolgreich aktualisiert",
    "reset-toast": "Benutzerdefinierte Farben für {theme} entfernt"
  },
}
```

##### 3. Add your file to the app

In [`./src/utils/languages.js`](https://github.com/Lissy93/dashy/tree/master/src/utils/languages.js), you need to do 2 small things:

First import your new translation file, do this at the top of the page.
E.g. `import de from '@/assets/locales/de.json';`

Second, add it to the array of languages, e.g:
```javascript
export const languages = [
  {
    name: 'English',
    code: 'en',
    locale: en,
    flag: '🇬🇧',
  },
  {
    name: 'German', // The name of your language
    code: 'de', // The ISO code of your language
    locale: de, // The name of the file you imported (no quotes)
    flag: '🇩🇪', // An optional flag emoji
  },
];
```
You can also add your new language to the readme, under the [Language Switching](https://github.com/Lissy93/dashy#language-switching-) section, and optionally include your name/ username if you'd like to be credited for your work. Done!

If you are not comfortable with making pull requests, or do not want to modify the code, then feel free to instead send the translated file to me, and I can add it into the application. I will be sure to credit you appropriately. 

# Adding a new option in the config file

This section is for, if you're adding a new component or setting, that requires an additional item to be added to the users config file.

All of the users config is specified in `./public/conf.yml` - see [Configuring Docs](./configuring.md) for info.
Before adding a new option in the config file, first ensure that there is nothing similar available, that is is definitely necessary, it will not conflict with any other options and most importantly that it will not cause any breaking changes. Ensure that you choose an appropriate and relevant section to place it under.

Next decide the most appropriate place for your attribute:
- Application settings should be located under `appConfig`
- Page info (such as text and metadata) should be under `pageInfo`
- Data relating to specific sections should be under `section[n].displayData`
- And for setting applied to specific items, it should be under `item[n]`

In order for the user to be able to add your new attribute using the Config Editor, and for the build validation to pass, your attribute must be included within the [ConfigSchema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.js). You can read about how to do this on the [ajv docs](https://ajv.js.org/json-schema.html). Give your property a type and a description, as well as any other optional fields that you feel are relevant. For example:

```json
"fontAwesomeKey": {
  "type": "string",
  "pattern": "^[a-z0-9]{10}$",
  "description": "API key for font-awesome",
  "example": "0821c65656"
}
```
or
```json
"iconSize": {
  "enum": [ "small", "medium", "large" ],
  "default": "medium",
  "description": "The size of each link item / icon"
}
```

Next, if you're property should have a default value, then add it to [`defaults.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/defaults.js). This ensures that nothing will break if the user does not use your property, and having all defaults together keeps things organised and easy to manage.

If your property needs additional logic for fetching, setting or processing, then you can add a helper function within [`ConfigHelpers.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigHelpers.js).

Finally, add your new property to the [`configuring.md`](./configuring.md) API docs. Put it under the relevant section, and be sure to include field name, data type, a description and mention that it is optional.  If your new feature needs more explaining, then you can also document it under the relevant section elsewhere in the documentation.

Checklist:
- [ ] Ensure the new attribute is actually necessary, and nothing similar already exists 
- [ ] Update the [Schema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.js) with the parameters for your new option
- [ ] Set a default value (if required) within [`defaults.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/defaults.js)
- [ ] Document the new value in [`configuring.md`](./configuring.md)
- [ ] Test that the reading of the new attribute is properly handled, and will not cause any errors when it is missing or populated with an unexpected value

---

## Updating Dependencies

Running `yarn upgrade` will updated all dependencies based on the ranges specified in the `package.json`. The `yarn.lock` file will be updated, as will the contents of `./node_modules`, for more info, see the [yarn upgrade documentation](https://classic.yarnpkg.com/en/docs/cli/upgrade/). It is important to thoroughly test after any big dependency updates.

---

## Developing Netlify Cloud Functions

When Dashy is deployed to Netlify, it is effectively running as a static app, and therefore the server-side code for the Node.js endpoints is not available. However Netlify now supports serverless cloud lambda functions, which can be used to replace most functionality.

#### 1. Run Netlify Dev Server

First off, install the Netlify CLI: `npm install netlify-cli -g`
Then, from within the root of Dashy's directory, start the server, by running: `netlify dev`

#### 2. Create a lambda function

This should be saved it in the [`./services/serverless-functions`](https://github.com/Lissy93/dashy/tree/master/services/serverless-functions) directory

```javascript
exports.handler = async () => ({
  statusCode: 200,
  body: 'Return some data here...',
});
```

#### 3. Redirect the Node endpoint to the function

In the [`netlify.toml`](https://github.com/Lissy93/dashy/blob/FEATURE/serverless-functions/netlify.toml) file, add a 301 redirect, with the path to the original Node.js endpoint, and the name of your cloud function

```toml
[[redirects]]
  from = "/status-check"
  to = "/.netlify/functions/cloud-status-check"
  status = 301
  force = true
```

---

## Hiding Page Furniture on Certain Routes
For some pages (such as the login page, the minimal start page, etc) the basic page furniture, (like header, footer, nav, etc) is not needed. This section explains how you can hide furniture on a new view (step 1), or add a component that should be hidden on certain views (step 2).

##### 1. Add the route name to the should hide array

In [`./src/utils/defaults.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/defaults.js), there's an array called `hideFurnitureOn`. Append the name of the route (the same as it appears in [`router.js`](https://github.com/Lissy93/dashy/blob/master/src/router.js)) here.

##### 2. Add the conditional to the structural component to hide

First, import the helper function:
```javascript
import { shouldBeVisible } from '@/utils/SectionHelpers';
```

Then you can create a computed value, that calls this function, passing in the route name:
```javascript
export default {
  ...
  computed: {
    ...
    isVisible() {
      return shouldBeVisible(this.$route.name);
    },
  },
};
```
  
Finally, in the markup of your component, just add a `v-if` statement, referencing your computed value
```vue
<header v-if="isVisible">
  ...
</header>
```

---

## Adding / Using Environmental Variables
All environmental variables are optional. Currently there are not many environmental variables used, as most of the user preferences are stored under `appConfig` in the `conf.yml` file.

You can set variables either in your environment, or using the [`.env`](https://github.com/Lissy93/dashy/blob/master/.env) file.

Any environmental variables used by the frontend are preceded with `VUE_APP_`. Vue will merge the contents of your `.env` file into the app in a similar way to the ['dotenv'](https://github.com/motdotla/dotenv) package, where any variables that you set on your system will always take preference over the contents of any `.env` file.

If add any new variables, ensure that there is always a fallback (define it in [`defaults.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/defaults.js)), so as to not cause breaking changes. Don't commit the contents of your `.env` file to git, but instead take a few moments to document what you've added under the appropriate section. Try and follow the concepts outlined in the [12 factor app](https://12factor.net/config).

---

## Building a Widget

### Step 0 - Prerequisites

If this is your first time working on Dashy, then the [Developing Docs](https://github.com/Lissy93/dashy/blob/master/docs/developing.md) instructions for project setup and running. In short, you just need to clone the project, cd into it, install dependencies (`yarn`) and then start the development server (`yarn dev`).

To build a widget, you'll also need some basic knowledge of Vue.js. The [official Vue docs](https://vuejs.org/v2/guide/) provides a good starting point, as does [this guide](https://www.taniarascia.com/getting-started-with-vue/) by Tania Rascia

If you just want to jump straight in, then [here](https://github.com/Lissy93/dashy/commit/3da76ce2999f57f76a97454c0276301e39957b8e) is a complete implementation of a new example widget, or take a look at the [`XkcdComic.vue`](https://github.com/Lissy93/dashy/blob/master/src/components/Widgets/XkcdComic.vue) widget, which is pretty simple.


### Step 1 - Create Widget

Firstly, create a new `.vue` file under [`./src/components/Widgets`](https://github.com/Lissy93/dashy/tree/master/src/components/Widgets).


```vue
<template>
<div class="example-wrapper">
</div>
</template>

<script>

import axios from 'axios';
import WidgetMixin from '@/mixins/WidgetMixin';
import { widgetApiEndpoints } from '@/utils/defaults';

export default {
  mixins: [WidgetMixin],
  data() {
    return {};
  },
  computed: {},
  methods: {
    fetchData() {
      // TODO: Make Data Request
    },
  },
};
</script>

<style scoped lang="scss">
</style>
```

All widgets extend from the [Widget](https://github.com/Lissy93/dashy/blob/master/src/mixins/WidgetMixin.js) mixin. This provides some basic functionality that is shared by all widgets. The mixin includes the following `options`, `startLoading()`, `finishLoading()`, `error()` and `update()`.
- **Getting user options: `options`**
	- Any user-specific config can be accessed with `this.options.something` (where something is the data key your accessing)
- **Loading state: `startLoading()` and `finishLoading()`**
	- You can show the loader with `this.startLoading()`, then when your data request completes, hide it again with `this.finishLoading()`
- **Error handling: `error()`**
	- If something goes wrong (such as API error, or missing user parameters), then call `this.error()` to show message to user
- **Updating data: `update()`**
	- When the user clicks the update button, or if continuous updates are enabled, then the `update()` method within your widget will be called

### Step 2 - Adding Functionality

**Accessing User Options**

If your widget is going to accept any parameters from the user, then we can access these with `this.options.[parmName]`. It's best to put these as computed properties, which will enable us to check it exists, is valid, and if needed format it. For example, if we have an optional property called `count` (to determine number of results), we can do the following, and then reference it within our component with `this.count`

```javascript
computed: {
  count() {
    if (!this.options.count) {
      return 5;
    }
    return this.options.count;
  },
	...
},
```

**Adding an API Endpoint**

If your widget makes a data request, then add the URL for the API under point to the `widgetApiEndpoints` array in [`defaults.js`](https://github.com/Lissy93/dashy/blob/master/src/utils/defaults.js#L207)

```javascript
widgetApiEndpoints: {
  ...
  exampleEndpoint: 'https://hub.dummyapis.com/ImagesList',
},
```

Then in your widget file:

```javascript
import { widgetApiEndpoints } from '@/utils/defaults';
```

For GET requests, you may need to add some parameters onto the end of the URL. We can use another computed property for this, for example:

```javascript
endpoint() {
  return `${widgetApiEndpoints.exampleEndpoint}?count=${this.count}`;
},
```

**Making an API Request**

Axios is used for making data requests, so import it into your component: `import axios from 'axios';`

Under the `methods` block, we'll create a function called `fetchData`, here we can use Axios to make a call to our endpoint.

```javascript
fetchData() {
  axios.get(this.endpoint)
    .then((response) => {
      this.processData(response.data);
    })
    .catch((dataFetchError) => {
      this.error('Unable to fetch data', dataFetchError);
    })
    .finally(() => {
      this.finishLoading();
    });
},
```

There are three things happening here:
- If the response completes successfully, we'll pass the results to another function that will handle them
- If there's an error, then we call `this.error()`, which will show a message to the user
- Whatever the result, once the request has completed, we call `this.finishLoading()`, which will hide the loader

**Processing Response**

In the above example, we call the `processData()` method with the result from the API, so we need to create that under the `methods` section. How you handle this data will vary depending on what's returned by the API, and what you want to render to the user. But however you do it, you will likely need to create a data variable to store the response, so that it can be easily displayed in the HTML.

```javascript
data() {
  return {
    myResults: null,
  };
},
```

And then, inside your `processData()` method, you can set the value of this, with:

```javascript
`this.myResults = 'whatever'`
```

**Rendering Response**

Now that the results are in the correct format, and stored as data variables, we can use them within the `<template>` to render results to the user. Again, how you do this will depend on the structure of your data, and what you want to display, but at it's simplest, it might look something like this:

```vue
<p class="results">{{ myResults }}</p>
```

**Styling**

Styles can be written your your widget within the `<style>` block.

There are several color variables used by widgets, which extend from the base pallete. Using these enables users to override colors to theme their dashboard, if they wish. The variables are: `--widget-text-color`, `--widget-background-color` and `--widget-accent-color`


```vue
<style scoped lang="scss">
p.results {
  color: var(--widget-text-color);
}
</style>
```

For examples of finished widget components, see the [Widgets](https://github.com/Lissy93/dashy/tree/master/src/components/Widgets) directory. Specifically, the [`XkcdComic.vue`](https://github.com/Lissy93/dashy/blob/master/src/components/Widgets/XkcdComic.vue) widget is quite minimal, so would make a good example, as will [this example implementation](https://github.com/Lissy93/dashy/commit/3da76ce2999f57f76a97454c0276301e39957b8e).


### Step 3 - Register

Next, import and register your new widget, in [`WidgetBase.vue`](https://github.com/Lissy93/dashy/blob/master/src/components/Widgets/WidgetBase.vue). In this file, you'll need to add the following:

Import your widget file
```javascript
import ExampleWidget from '@/components/Widgets/ExampleWidget.vue';
```

Then register the component
```javascript
components: {
  ...
  ExampleWidget,
},
```

Finally, add the markup to render it. The only attribute you need to change here is, setting `widgetType === 'example'` to your widget's name.
```vue
<ExampleWidget
  v-else-if="widgetType === 'example'"
  :options="widgetOptions"
  @loading="setLoaderState"
  @error="handleError"
  :ref="widgetRef"
/>
```

### Step 4 - Docs

Finally, add some documentation for your widget in the [Widget Docs](https://github.com/Lissy93/dashy/blob/master/docs/widgets.md), so that others know hoe to use it. Include the following information: Title, short description, screenshot, config options and some example YAML.


**Summary**: For a complete example of everything discussed here, see: [`3da76ce`](https://github.com/Lissy93/dashy/commit/3da76ce2999f57f76a97454c0276301e39957b8e)
