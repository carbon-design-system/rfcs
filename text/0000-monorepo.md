- Start Date: 2019-04-XX
- RFC PR: (leave this empty)
- Carbon Issue: (leave this empty)

# Motivation

Carbon is currently split up into a variety of projects that are maintained by the core team and the Carbon community. This list includes:

- `carbon-components` for component styles and Vanilla implementation
- `carbon-components-react` for React components
- `carbon-components-angular` for Angular
- `@carbon/vue` for Vue
- `carbon-addons-*` for Business-unit specific components built alongside Carbon

_Note: this list does not include projects like `carbon-website` or the Carbon Elements monorepo_

Organizing projects in this way has naturally lead to:

- Autonomy of each group to make decisions on how to best deliver their work
- Focused contributions for specific implementations, which can make first-time contributions less intimidating
- Tailored backlogs for each project, and potentially repo-specific labelling systems that work best for maintainers

Some of the drawbacks of this organization include:

- Low amount of code re-use between projects
- Lack of consistency in projects with respect to:
  - Project tasks
    - Formatting
    - Linting
    - Testing (if sharing same testing framework)
  - `package.json`
    - Meta information
    - Tags
    - `main`/`module` distinctions
  - Package output
    - Bundle types
    - Is tree-shakeable?
    - Can I use only one component?
- Varying versions between the implementations (v10 vs v7 vs v3 vs v1)
- All frameworks have trouble working on styles and new functionality because of separation
- Sometimes it can be confusing where to make an issue, e.g. is it a style bug or implementation-specific?





This RFC proposes a new structure for the Carbon ecosystem in the form of a core project used for all components, their implementations, and tooling related to building components.

- Build components like Reach in that you can use them OOTB without styling, but can also include carbon styling
  - Built on top of theme engine

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
