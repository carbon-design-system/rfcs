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

If publishing through GitHub pages, we would most likely need to share the same vanity URL and have packages published at sub-paths. A couple of options could be:

```
https://www.carbondesignsystem.com/storybook/<framework>
https://docs.carbondesignsystem.com/<framework>
```

## Component packages

In theory, we could support component packages for every framework implementation. As a result, the following packages would be siblings of each other:

```
packages
├── button
├── button-angular
├── button-react
└── button-vue
```

However, we could limit these packages to only core packages, if needed. We could also move components to their own sub-packages folder, for example `packages/react` or a top-level `react` set of packages.

The primary component package would be found under `packages/button` and would contain the following contents:

- The Vanilla.js implementation of the component, this is to facilitate sharing with frameworks
- The style representation of the component, initially this would be Sass but could in theory be multiple formats that are offered
- A specification for the component (where applicable)
- Metadata around the component (could be a generated artifact) to be consumed by websites to aid in search result ranking
- Tests for the component
- A development "story" for storybook
- Other exports that could be useful for sites
  - Usage content for the component (mdx)
  - Component docs (mdx)

The package name itself would follow the convention `@carbon/<component>`, for example `@carbon/button`. Each component is given it's own package to allow for the following situations:

- Ability to see usage information around a specific component
- Allow add-on teams to import components as needed (see [Community-contributed components](#community-contributed-components))
- Allow ability to use individual components as-needed
- Allow framework-specific solutions to easily add in components and re-export them as-needed

### Frameworks

A framework-specific component package would be found under `packages/button-<framework>`. For example, `packages/button-react`. This package would import `@carbon/button` (found at `packages/button`) and has the option to use the Vanilla.js implementation if applicable.

The framework-specific package would include:

- The framework implementation of the component, using Vanilla as needed
- A specification for the component
- Metadata for the component
- Tests
- A development "story" for the component
- Imports styles from `@carbon/button` and re-exports them

We include **both** styles and implementation so that instead of having to do:

```bash
yarn add @carbon/button @carbon/button-react
```

One can write:

```bash
yarn add @carbon/button-react
```

As it would contain everything needed to use the component, while still relying on the core package and receiving updates.

In addition, each framework would have an entrypoint for core components. For React, this package would be found under: `packages/react` and would be published to `@carbon/react`.

The `@carbon/react` package would be responsible for depending on all core components, and re-exporting their implementations and styles. This means that in order to use the React set of carbon components, one would only need to install `@carbon/react` as it includes both component implementations and styles.

### Community-contributed components

In addition to core components and framework implementations, this proposal also looks to introduce the idea that there is **one** repo where carbon component development lives. This idea drifts from the add-on model and is explored in the [add-ons](#add-ons) section of this RFC.

With respect to the packages, they would follow the same structure as above but with (most likely) less information around usage, documentation, and component specs.

### Component development

We would structure the project so that component development happened using storybook. Given our usage of Yarn workspaces and lerna, we could subsequently build packages for each storybook development environment for each implementation. In addition, we could have specific frameworks have their own development environment, where applicable.

Given the potential size of the component library, it may also be necessary to invest in development tooling to help make working on individual components fast and simple. One idea could be an internal CLI tool that would enable folks to target specific frameworks or components for development in a storybook environment. This could look like:

```bash
# Develop a component in a vanilla environment
carbon-cli develop @carbon/button

# Develop a component in a react environment
carbon-cli develop @carbon/button-react

# Develop multiple components
carbon-cli develop @carbon/button-react @carbon/pagination-react @carbon/data-table-react

# Develop all react components
carbon-cli develop @carbon/react
```

### Component package files

In addition to hosting files relevant to implementation of a component, each package would have relevant files kept in sync or generated automatically. These types of files include:

- READMEs
  - Can automate the structure/format, including dynamic documentation and example links while allowing custom usage sections
- npm files
  - `package.json` will have deterministic ordering and appropriate fields for versions, keywords, and publish config
  - `.npmignore` will have appropriate defaults
- A `docs` folder for generated SassDoc and JSDoc information

## Ownership and access

## Add-ons

## Misc

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
