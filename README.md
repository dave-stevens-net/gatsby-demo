# Gatsby Demo

Gatsby is a static site generator. You can source your pages from various data sources, such as a CMS, Markdown or JSON, etc.

### Setup

```
npm install -g gatsby-cli  # Install the Gatsby CLI
gatsby new gatsby-demo     # Create a Gatsby site
cd gatsby-demo
npm start                  # Start the development server with hot reloading
npm run build              # Build out the static pages
npm run serve              # Serve up the static pages with a web server
```

### Testing Your New Site

Visit http://localhost:8000/ and verify that it is working.
Visit http://localhost:8000/___graphql to run GraphiQL, which is an in-browser tool for writing and testing GraphQL queries

What is GraphQL, you may be wondering. GraphQL was developed by Facebook.

> It provides an efficient, powerful and flexible approach to developing web APIs, and has been compared and contrasted with REST and other web service architectures. It allows clients to define the structure of the data required, and exactly the same structure of the data is returned from the server, therefore preventing excessively large amounts of data from being returned, but this has implications for how effective web caching of query results can be. - from https://en.wikipedia.org/wiki/GraphQL

With Gatsby you use GraphQL to gather only the data you need from the source data in order to construct a page. Each template defines a React component for the page and exports a graphQL query that the page uses.

GraphiQL is an in-browser tool that is available only when running gatsby's development server. This can be useful for testing out the GraphQL queries that you define in your templates.

### Sourcing Pages from Markdown

### Linking Pages
