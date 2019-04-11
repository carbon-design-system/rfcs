# Sass imports

- Start Date: (fill me in with today's date, 2019-04-05)
- RFC PR: (leave this empty)
- Carbon Issue: (leave this empty)

# Summary

Currently the way individuals can include carbon styles is through the path:

```
carbon-components/scss/globals/scss/styles.scss
```

In addition, individuals can include individual components with the path:

```
carbon-components/scss/components/<component>/<component>.scss
```

As we start to add more to our system, like inline theming, some of these paths become awkward. For example, consider our theming case:

```scss
@import 'carbon-components/scss/globals/scss/theme';

$carbon--theme: $carbon--theme--g100;

@import 'carbon-components/scss/globals/scss/styles';
```

This RFC aims to propose a simplification of our Sass imports, in addition to aligning the standards set by Carbon Elements for how to ship sass files.

# Detailed design

The entrypoint for `carbon-components`'s SCSS would be updated to:

```scss
@import 'carbon-components/scss/styles.scss';
```

_Note: all new import paths will not remove the old import path, but instead will serve as an alias_

Specific componene imports would stay as:

```scss
@import 'carbon-components/scss/components/button/button';
```

We would also expose element-specific behavior:

```scss
@import 'carbon-components/scss/colors';
@import 'carbon-components/scss/themes';
@import 'carbon-components/scss/layout';
@import 'carbon-components/scss/grid';
@import 'carbon-components/scss/icons';
@import 'carbon-components/scss/type';
@import 'carbon-components/scss/motion';

// Maybe these would be useful, as well?
@import 'carbon-components/scss/spacing';
@import 'carbon-components/scss/reset';
```

Alongside any helpers that they may want to reference:

```scss
@import 'carbon-components/scss/variables';
@import 'carbon-components/scss/mixins';
@import 'carbon-components/scss/functions';
```

We could also expose hidden APIs/features, like feature flags:

```scss
@import 'carbon-components/scss/feature-flags';
// Manage config for carbon, could also be settings
@import 'carbon-components/scss/config';
@import 'carbon-components/scss/settings';
```

## Common use-cases

#### Importing all carbon styles

```scss
@import 'carbon-components/scss/styles.scss';
```

#### Import specific component styles

```scss
@import 'carbon-components/scss/components/<component>/<component>';
```

#### Import only components needed

```scss
@import 'carbon-components/scss/components/accordion/accordion';
@import 'carbon-components/scss/components/button/button';
```

## Removals from public entrypoint

```scss
$css--body: true !default;
$css--reset: true !default;
$css--helpers: true !default;

// Do we need this anymore?
$css--use-layer: true !default;

// Remap to type classes?
$css--typography: true !default;

// Plex is default now
$css--plex: true !default;
// No longer emit font-face
$css--font-face: true !default;
```

# Drawbacks

- Creates a work item for every team using our current Sass styles entrypoint

# Alternatives

No alternatives as this a proposal for an existing norm that matches elements.

# Adoption strategy

- Could create migration helpers using `carbon-upgrade` to rewrite paths automatically for teams
- Communicate roadmap and decision through blogs and on slack
- Communicate when work is merged in and link to blogs and migration tool

# How we teach this

We should document these as part of the release in SassDoc, in the project README, and on the site.

# Unresolved questions
