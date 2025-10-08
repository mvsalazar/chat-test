# chat-test

## Getting Started

Install dependencies:

```
yarn install
```

Start the Vite dev server (defaults to `http://localhost:5173`):

```
yarn dev
```

Create a production build:

```
yarn build
```

Preview the production output locally:

```
yarn preview
```

Lint source files:

```
yarn lint
```

## Testing

Automated tests are not configured for this project. Verify changes manually by running the dev server (`yarn dev`) or building the production bundle (`yarn build`) and exercising key flows in the browser.

## Deployment

1. Build the project: `yarn build`
2. Deploy the contents of the generated `dist/` directory to your static host of choice (e.g., Netlify, Vercel, or S3 + CloudFront).
