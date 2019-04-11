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
```

# Drawbacks

Why should we *not* do this? Please consider:

- implementation cost, both in term of code size and complexity
- whether the proposed feature can be implemented in user space
- the impact on teaching people Carbon
- integration of this feature with other existing and planned features
- cost of migrating existing Carbon applications (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Adoption strategy

If we implement this proposal, how will existing Carbon developers adopt it? Is
this a breaking change? Can we write a codemod? Should we coordinate with
other projects or libraries?

# How we teach this

What names and terminology work best for these concepts and why? How is this
idea best presented? As a continuation of existing Carbon patterns?

Would the acceptance of this proposal mean the Carbon documentation must be
re-organized or altered? Does it change how Carbon is taught to new developers
at any level?

How should this feature be taught to existing Carbon developers?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?
