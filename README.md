# Vuse

Advanced page/email builder based on [Vue.js](https://vuejs.org/).

## What is this?

This builder (sections builder) reuses your Vue components as **editable sections** to produce an interactive page builder to the end user, you can use it as a prototyping tool as well as it is sort-of a block builder.

The user/developer can then export the builder for usability in multiple formats, the following are the officially supported ones:

- `json` A json object which can be later used to re-render a page, particularly useful if you plan to have dynamic pages or want to store them in a Database.
- `preview` opens a new page without the editable logic in new tab to see the end result.
- `pwa` produces a zip file which contains all the page files and images neatly packed, this is probably the format you will use for page/prototype landing page builder, The project is augmented by default with service workers to support offline viewing.

The builder is just a Vue plugin, so you can integrate this into your own projects without needing to create separate bundles for it.

## Installation

### Package managers

First step is to install it using `yarn` or `npm`:

```bash
npm install vuse

# or use yarn
yarn add vuse
```

### CDNs

Or add it as a script tag in your projects.

- [unpkg](https://unpkg.com/vuse)

```html
<script src="https://unpkg.com/vue@2.4.2"></script>
<script src="https://unpkg.com/vuse"></script>
```

### Usage

> **If you added it using a script tag, you can skip this section. as it will be auto-installed for you**

```js
import Builder from 'vuse';

Vue.use(Builder);
```

You can start using the `b-builder` component to build things now.

This package does not include any sections. The builder is just a system of helpers for managing customizable sections, seeding fake data, exporting views and re-rendering them. The logic for your components/sections is written by you eventually. **we will be adding a huge number of sections soon to be used with this one**. You can use the included examples after reading the API to build your own for now.

## API

<!-- > Examples use [Base.framework](https://github.com/baianat/base.framework) design system for grid and UI elements. -->
> Examples use [pug](https://pugjs.org) template language to make it easier to work with templates.

### Injected properties and methods

Your section components will have some properties injected to help you customize their behavior in building phase and production/render phase.

#### $section

An instance of the [section class](https://github.com/baianat/vuse/blob/master/src/js/section.js) that represents this component.

#### $builder

An instance of the singleton [builder class](https://github.com/baianat/vuse/blob/master/src/js/builder.js)

#### $sectionData

Is a computed property that mirrors `$section.data` which contains the current values (text, images, etc...) for the section.

### Section

A section is the building block of the page, below is an example of a header section.

```pug
<template lang="pug">
  section.header(
    v-styler="$sectionData.classes"
    :class="[{'is-editable': $builder.isEditing}, $sectionData.classes]"
  )
    .container
      .grid
        .column.is-screen-6.add-center-vertical
          h3.header-title(
            :class="{'is-editable': $builder.isEditing}"
            v-html="$sectionData.title"
            v-styler="$sectionData.title"
          )
          p.header-content(
            :class="{'is-editable': $builder.isEditing}"
            v-html="$sectionData.content"
            v-styler="$sectionData.content"
          )
          a.button(
            :class="[{'is-editable': $builder.isEditing}, $sectionData.button.classes]"
            :href="$sectionData.button.href"
            v-html="$sectionData.button.text"
            v-styler="$sectionData.button"
          )
        .column.is-screen-6
          uploader(
            class="header-image"
            path="$sectionData.images[0]"
          )
</template>

<script>
  import { types } from 'vuse';

  export default {
    name: 'hero1',
    cover: '../covers/hero1.png',
    $schema: {
      title: types.Title,
      content: types.Text,
      images: [types.Image],
      button: types.Button,
      classes: types.ClassList
    },
    props: {
      id: Number
    }
  };
</script>
```

Each section has several elements that can be edited. Section data are stored in `$sectionData` object which is reactive.

### Adding the ability to edit elements in a section

1. Add `is-editable` class to it. Since editable state can be toggled off/on, it's always good to bind `is-editable` class to change when editing mode changes. e.g. `:class="{'is-editable': $builder.isEditing}"`
1. Add [`v-styler`](https://github.com/baianat/builder#v-styler) directive to the element
1. Bind the element’s innerHTML with its equivalent data e.g. `v-html="$sectionData.button.text"`
1. If you have any other data that `v-styler` changes, you have to bind it too. e.g. `:href="$sectionData.button.href"`

Putting it all together

```html
  <a
    :class="[{'button', 'is-editable': $builder.isEditing}, $sectionData.button.classes]"
    :href="$sectionData.button.href"
    v-html="$sectionData.button.text"
    v-styler="$sectionData.button"
  ></a>
```

After creating the HTML structure, you should configure the section schema to use the built-in seeder to provide you with initial/fake values when the component is instantiated in build/edit mode. Or you can set the initial values yourself instead of seeding them.

```html
<script>
  import { types } from 'vuse';

  export default {
    // section name
    name: 'hero1',
    // section cover image
    cover: '../cover-hero1.png',
    // group multiple sections
    group: 'heros',
    // section data schema
    $schema: {
      // main title
      title: types.Text,
      // main content
      content: types.Text,
      // section classes
      classes: types.ClassList,
      // array of section's images
      images: [
        types.Image,
        types.Image
      ],
      // object holds button data, href etc..
      button: types.Button,
      // if section has many columns you can use
      // a columns array to separate each column's data
      columns: [
      {
        title: types.Title,
        content: types.Text,
        group: types.group
      },
      {
        title: types.Title,
        group: types.group
      }
    ]
    },
    props: {
      id: Number // it is required to have an id prop.
    }
  };
</script>
```

### v-styler

This directive is automatically injected in your section components, and can be used to facilitate editing elements greatly, since it has support for multiple element types like `div`, `a`, `button` and `p` tags as well.

To tell styler which variable to update, you pass it as directive expression e.g. `v-styler="$sectionData.button"`

The styler directive has four types `text`, `button`, `section` or `grid`. By default, the directive can know the type implicitly, from the element tag or from the provided schema. 

If you want to explicitly specify the type, you can pass it as a directive modifier e.g. `v-styler.button="$sectionData.button"`.

### Using the section

Until now, we have only been creating our section component, we now need to introduce it to our builder so it can use it:

```js
import Builder from 'vuse';
import section from './components/sections/section';

// Builder has Vue-like API when registering components.
Builder.component(section);
Vue.use(Builder);

new Vue({
  el: '#app'
});
```

```html
<div id="app">
  <b-builder></b-builder>
</div>
```

You only have to register the component on the Builder plugin, which has a Vue-like API. This ensures your Vue global scope isn't polluted by the builder and keeps everything contained within the `b-builder` component.

### Exporting

There are three ways to export the built page: preview, pwa or json. When clicking the save button the `b-builder` component emits a saved event with the builder instance as its payload, which exposes an `export` method.

```html
<div id="app">
  <b-builder @saved="onSave"></b-builder>
</div>

<script>

  new Vue({
  el: '#app',
  methods: {
    onSave (builder) {
      // you can use 'preview', 'pwa' or 'json' strings
      // 'json' is the default exporting mode
      builder.export('preview');
    }
  }
});
</script>
```

## License

MIT
