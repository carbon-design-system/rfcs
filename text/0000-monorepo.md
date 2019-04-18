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

Given that this project would be used for all carbon development, we would need to look at how we're currently doing ownership and access across our existing projects and use that to determine what would be most appropriate for this project.

At a high-level, we would use GitHub's [`CODEOWNERS`](https://help.github.com/en/articles/about-code-owners) functionality to assign GitHub teams to specific parts of the project. These teams would be aliases of our workstreams, most likely. This would trigger required reviews

Push access to the project would be given to all members of the GitHub teams included in the project.

Publish access would ideally be automated through CI, but manual access can be granted to packages that belong to a specific GitHub team / workstream.

## Add-ons

Within this proposal, the role of an add-on will most likely need to be re-defined. Currently, an add-on exists as a supplement to the Carbon Design System. This means that teams arrive at solutions that they need that don't exist in Carbon currently, and they have a desire to share within their team. The add-on model works well for this direct use-case, but unfortunately when scaling out Carbon to all of IBM this model has a habit of entrenching behavior.

For example, imagine we had teams A, B, and C all working on platforms. Team A comes up with a `PlatformHeader` component that teams B and C would like to use.

Team B can use the component as-is, which is great. However, Team C would like some current functionality tweaked to meet their use-case. While teams A and C could work together to arrive at a joint resolution, often times deadlines and scope limit this kind of collaboration.

In addition, the experience of the developer can be confusing in these moments. The packages for these add-ons would like be:

```
# Team A
carbon-addons-a
carbon-addons-b
carbon-addons-c
```

In the existing landscape, team B would need to install (at minimum) the following:

```bash
yarn add carbon-components \
  carbon-components-react \ 
  carbon-icons \
  carbon-addons-a \
  carbon-addons-a-react \
  carbon-addons-b \
  carbon-addons-b-react 
```

Ideally, the best experience for this group would be a single `carbon-addons-b` or `carbon-addons-b-react`.

The other downside to this is the aspect of who owns support and who decides what is in-scope for enhancements of a component. When components are built inside add-ons, this can fall on the team whose add-on houses the component. However, if another team is requesting enhancements most likely they should be the ones to implement (when reasonable).

As a result, the proposal is that **all** component development happens in the Carbon repo and add-on packages move to becoming **the defacto way to use Carbon for a Business unit, portfolio, offering, or team**. What this means is that add-ons move increasingly towards their domain, and components like `PlatformHeader` should move to the main Carbon repo.

This does not prohibit multiple `PlatformHeader` components, however. If we used the situation above, we could have a singular `platform-header` package. Inside this package we could have multiple implementations under `PlatformHeader.A.js`, `PlatformHeader.B.js`, etc. This allows us to both allow teams to ship things under their unique deadlines/constraints while also allowing colocation of related components with the eventual goal of converging implementations.

In addition, having shared packages allows teams to make proposals through a formal RFC process and decide on changes as a group instead of tying ownership to any particular add-on.

As a result, the add-on packages increasingly move towards providing Carbon in the best way for their group. This can include depending on the entire component library, or only on the components a brand needs. In addition, add-ons would provide domain-specific behavior (like data fetching) alongside components to make it even easier for product developers to deliver new functionality.

# Drawbacks

The main disadvantages of using the strategies outlined above is that it forces our model into a monorepo. While this in and of itself is not a negative thing, it does shift a portion of our development time towards investing in tooling to help make working in a monorepo more enjoyable.

Aspects of this include a custom CLI to help with working on a single component, in addition to tooling or strategies for scaling out our core tools so that they are either fast on a large number of files, or only run on files that have chanegd.

In addition, while moving work to one project eliminates any confusion to issue creation, we run into a problem where everyone will get additional notifications that may not be related to their core project. The immediate action after this would be to ignore conversations that one is not actively participating in, but this could lead to issues not getting any response.

There are ways to mitigate this, including issue templates, but ultimately scaling issues and pull requests in a monorepo with multiple teams/collaboratos is still an unknown.

Finally, there is a concern around making everything under the Carbon umbrella public. The main use-case being: what if a team wants to use Carbon to build a component but is unable to make this component open-source? One solution could be the mirror our project to an internal GitHub Enterprise repo, but this could cause fragmentation in the ecosystem.

# Alternatives

The main alternative would be for **only** Carbon core to have a monorepo design. Given a desire to want to ship per-component packages, it is hard to see how we could scale doing repo-per-compnoent efforts. Particularly with respect to consistency between projects, keeping things up-to-date, and the experience of working with interlinked components, aka compound components.

# Adoption strategy

The majority of this work would be internal to the Carbon core team, or for the maintainers of specific frameworks and add-ons.

The ideal would be to break up this proposal into several stages that can then help determine a ship/no-ship status for this approach. These stages include:

- Stage 1: Convert existing carbon core projects to a single monorepo (the Carbon repo)
- Stage 2: Add framework-specific projects to the Carbon repo
- Stage 3: Add add-on components to the Carbon repo

Alongside these stages, we would need to define several items including:

- What is the process for adding a new component package? Is it okay if this package lives only for a particular framework?
- What is the release cadence like for packages?
- How do we communicate when components are core-supported versus community-supported? In their README? On the website?
- How does one suggest another framework implementation? For example, if a team wanted preact or svelte.

However, it should be noted, that each stage is setup to provide feedback as to whether or not to proceed. If we go through stage 1 and decide the strategy **does not** work, then we can stop any of the subsequent stages to prevent further churn in the ecosystem.

# How we teach this

The top-level points we need to communicate will be:

- Carbon is moving towards a single repo (or potentially two if doing private) where **all** component development occurs
- Add-ons are being changed in scope to become how a team uses Carbon in their Business Unit, offering, portfolio, or team and uses component packages alongside domain or tool-specific requirements for their group
- Components in the Carbon repo can be core or community-supported, and how to distinguish between them
- How does one develop inside of the Carbon repo on a single component, a group of components, a pattern, etc.

# Unresolved questions

- Where do community-lead packages live with respect to Design Kits?
