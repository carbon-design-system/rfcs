- Start Date: 2019-04-XX
- RFC PR: (leave this empty)
- Carbon Issue: (leave this empty)

# Overview

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
- Low amount of code re-use between projects
- Varying versions between the implementations (v10 vs v7 vs v3 vs v1)
- All frameworks have trouble working on styles and new functionality because of separation
- Sometimes it can be confusing where to make an issue, e.g. is it a style bug or implementation-specific?
- Coordinating major releases of the system is challenging

This RFC proposes a new structure for the Carbon ecosystem in the form of a core project used for all components, their implementations, and tooling related to building components. This new structure would provide the following benefits for the ecosystem:

- All components are available as standalone packages
- Implementation-specific packages now supply everything needed to have a project running with carbon (no need to install `carbon-components`!)
- Versions across packages follow the same major version, eliminating confusion for what support is provided for a given component or implementation
- Increase visibility as to what is a carbon component
- Colocation of components facilitates consistency and hopefully code re-use
- Increase quality of each component in the ecosystem

# Detailed design

## The monorepo

This proposal frames Carbon as one public repo where all components and their implementations live. This repo could live under IBM/carbon, or carbon-design-system/carbon.

The repo itself will be bootstrapped using a combination of [Yarn Workspaces](https://yarnpkg.com/lang/en/docs/workspaces/) and [Lerna](https://lernajs.io). We will configure yarn to have an offline mirror at `.yarn-offline-mirror` to enable deployments in CI without having to touch the live registry.

At the top-level of the project, the folder structure would roughly look like:

```
.
├── .yarn-offline-mirror
├── .yarnrc
├── lerna.json
├── package.json
├── packages
└── yarn.lock
```

All components would live under the `packages` folder as standalone component packages. We would also expose packages for interacting with styles and implementation-specific bundles of components or patterns. At a high-level, this would look like:

```
packages
├── button
├── button-react
├── button-vue
├── components
├── react
├── storybook-html
├── storybook-react
├── storybook-vue
├── styles
└── vue
```

_Note: the current proposal has components from different implementations living together, but that is not a direct goal of this RFC. We just wanted to show how we could support multiple frameworks in the same project._

### Yarn and Lerna

The project itself will be managed as a Yarn workspace, which allows us to integrate multiple types of packages into the same project successuflly. In other words, we could use this setup to develop components in Angular, Vue, React, etc. through the options given to us by using Yarn (though we do not have to do this).

The counterpart to managing multiple packages is [Lerna](https://lerna.js.org/). This will enable us to have consistent package publishing both in CI for releases and in manual publishing situations.

### Project settings

At the top-level, the project will be opinionated about:

- File formatting with prettier
- Linting with ESLint and Stylelint
- Testing with Jest
- `.editorconfig` settings

However, this does not mean that all packages in the project have to follow these rules. If needed, packages can opt-out of these and use their own settings that would be more appropriate.

### Continuous Integration

The project will use CircleCI for continuous integration with a Node.js docker base image. CircleCI has been a good fit so far, and we anticipate being able to use its [workflows](https://circleci.com/docs/2.0/workflows/) functionality to help maximize parallelism for ci checks across packages.

In addition, CircleCI can allow us to automatically publish canaries for each package, alongside coordinate automated releases through native git behavior or custom rules that we define. It will also enable continuous deployments for development environments, like published storybooks.

#### Publishing artifacts

If artifacts are necessary for component or implementation packages, like storybook or examples, then we can automate this through CircleCI to publish to either IBM Cloud or GitHub pages.

- Public vs private
- Project
- Dependencies
- Development
- CI
- Publishing

- CODEOWNERS
- All components in package
- Talk structure

Questions:

- how does a new package (component) get added?
- how do we communicate support for a package?
- issues
  - how to appropriately tag
- pull requests
  - who owns reviews?
- notifications explosion?

## Component packages

## Framework packages

## Patterns

## Add-ons

## Misc

- Spec
- tooling
- Storybook

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
