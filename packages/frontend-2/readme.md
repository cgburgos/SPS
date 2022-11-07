# frontend-2 🏬

The replacement for our old Vue 2 Vue CLI frontend SPA app. This one's built with Vue 3, Nuxt 3, Tailwind and is server side rendered. Should be a faster and nicer experience both for our devs and our users!

Look at the [nuxt 3 documentation](https://v3.nuxtjs.org) to learn more.

## TODO

- Validate client-side build, it seems to over-bundle lodash and potentially some other things

## Setup

Make sure to install the dependencies:

```bash
yarn install
```

And create an `.env` file from `.env.example`.

## Development

Start the development server on http://localhost:8081

```bash
yarn dev
```

### Typed GraphQL

Type your queries & fragments using the `graphql()` helper from `~~/lib/common/generated/gql` and then run `yarn gqlgen` (or `yarn gqlgen:watch` to run it in watch mode) to generated TS typing information for these GQL documents.

More info: https://the-guild.dev/blog/unleash-the-power-of-fragments-with-graphql-codegen

## Production

Build the application for production:

```bash
yarn build
```

You can serve the production build locally by running `yarn preview` afterwards.

Checkout the [deployment documentation](https://v3.nuxtjs.org/guide/deploy/presets) for more information.