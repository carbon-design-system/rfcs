- Start Date: 2018-10-16
- RFC PR: (leave this empty)
- Carbon Issue: (leave this empty)

# Summary

Component selectors are intended to be JavaScript-specific bindings for
selectors in the `carbon-components` codebase. This will allow JavaScript
implementations to import selectors directly from `carbon-components` instead of
having to hard-code selectors.

# Basic example

As an initial example, let's use the `accordion` component. Inside of
`src/components/accordion` we will add a `selectors.js` file that looks like:

```js
import prefix from '../../globals/js/prefix';

export const accordion = `${prefix}--accordion`;
export const arrow = `${prefix}--accordion-arrow`;
export const content = `${prefix}--accordion-content`;
export const heading = `${prefix}--accordion-heading`;
export const item = `${prefix}--accordion-item`;
export const title = `${prefix}--accordion-title`;
```

Inside of the corresponding component in the React.js implementation we could do
the following:

```jsx
import { accordion } from 'carbon-components/umd/selectors';
import PropTypes from 'prop-types';
import React from 'react';
import classnames from 'classnames';

const Accordion = ({ children, className, ...other }) => {
  // We also have accordion.arrow, accordion.content, etc. available here
  const classNames = classnames(accordion.accordion, className);
  return (
    <ul
      className={classNames}
      role="tablist"
      aria-multiselectable="true"
      {...other}>
      {children}
    </ul>
  );
};
```

# Motivation

Exposing component selectors allows all implementation libraries to directly
consume our selectors without having to hard-code any values. This will enable
core team members to make changes to selector names, in addition to helping
migrations to updated selectors in the future.

# Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody
familiar with Carbon to understand, and for somebody familiar with the
implementation to implement. This should get into specifics and corner-cases,
and include examples of how the feature is used. Any new terminology should be
defined here.

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
