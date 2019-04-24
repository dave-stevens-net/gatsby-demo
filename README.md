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

1. Install markdown plugin.
    ```
    npm i gatsby-transformer-remark
    ```
1. Rename the `src/pages/index.js` to `src/pages/index-old.js` since we will be creating our home page from markdown.
1. Create the `src/templates` and `src/data` folders
1. Code a `src/data/home.md` data source file as follows.
    ```
    ---
    path: "/"
    date: "2017-11-07"
    title: "Home Page Title"
    ---
    This is the content of the page.

    * This is list item 1
    * This is list item 2
    ```
1. Code a `src/templates/home.js` template component.
    ```
    import React from "react"
    import { graphql } from "gatsby"

    export default function HomePageTemplate({
        data, // this prop will be injected by the GraphQL query below.
    }) {
        const { markdownRemark } = data // data.markdownRemark holds our post data
        const { frontmatter, html } = markdownRemark
        return (
            <div className="blog-post-container">
                <div className="blog-post">
                    <h1>{frontmatter.title}</h1>
                    <h2>{frontmatter.date}</h2>
                    <div
                        className="blog-post-content"
                        dangerouslySetInnerHTML={{ __html: html }}
                    />
                </div>
            </div>
        )
    }

    export const pageQuery = graphql`
        query($path: String!) {
            markdownRemark(frontmatter: { path: { eq: $path } }) {
                html
                frontmatter {
                    date(formatString: "MMMM DD, YYYY")
                    path
                    title
                }
            }
        }
    `
    ```
1. Add the following configuration to the bottom of your `gatsby-config.js` file in your root folder. The first entry adds the filesystem data source called `markdown-pages`. The 2nd entry adds the plugin. The plugin looks for a data source called `markdown-pages` for the markdown files.
    ```
    plugins:[
        ...
        // Only add the following two elements within the plugins array.
        {
            resolve: `gatsby-source-filesystem`,
            options: {
                path: `${__dirname}/src/data`,
                name: "markdown-pages",
            },
        },
        `gatsby-transformer-remark`
    ],
    ```
1. Add the following to your `gatsby-node.js` file in your root folder. The `gatsby-node.js` file is only run during the build process to build out your pages.
    ```
    const path = require("path")

    exports.createPages = ({ actions, graphql }) => {
        const { createPage } = actions

        const homeTemplate = path.resolve(`src/templates/home.js`)

        return graphql(`
            {
                allMarkdownRemark(
                    sort: { order: DESC, fields: [frontmatter___date] }
                    limit: 1000
                ) {
                    edges {
                        node {
                            frontmatter {
                                path
                            }
                        }
                    }
                }
            }
        `).then(result => {
            if (result.errors) {
                return Promise.reject(result.errors)
            }

            result.data.allMarkdownRemark.edges.forEach(({ node }) => {
                console.log(`Building page ${node.frontmatter.path}...`)
                createPage({
                    path: node.frontmatter.path,
                    component: homeTemplate,
                    context: {}, // additional data can be passed via context
                })
            })
        })
    }

    ```
    
Run and test out your application to make sure the home page now shows your markdown page.

Now let's add a new template called `basic`.
    
1. Add a new front matter field called `template` to the existing `src/data/home.md` so that it looks like the following:
    ```
    ---
    path: "/"
    date: "2017-11-07"
    title: "Home Page Title"
    template: "home"
    ---
    ```
1. Then copy the home markdown page to `src/data/contact.md` and modify the data to be a contact page. Be sure to change the path to `/contact` and the template to `basic`. the front matter section should look like...
    ```
    ---
    path: "/contact"
    date: "2017-11-07"
    title: "Contact Page Title"
    template: "basic"
    ---
    ```
1. Then copy the home template page to `src/templates/basic.js` and make a modification like adding a note at the bottom that template is called basic.

Now if you start Gatsby you will notice that both the http://localhost:8000 and http://localhost:8000/contact work and show their respective pages. However, the contact page is still showing the home templates. That's because the build process (`gatsby-node.js`) has not been improved to pull the template dynamically from the source data.

To do this we will need to replace the `homeTemplate` variable with an object map called `templates` as follows.
```
    const templateNames = ['home', 'basic']
    const templates = {}
    templateNames.forEach(templateName => {
        templates[templateName] = path.resolve(`src/templates/${templateName}.js`)
    })
```

Then we need to remember to add the template to our graphql query inside the `gatsby-node.js` file.
```
    return graphql(`
        {
            allMarkdownRemark(
                sort: { order: DESC, fields: [frontmatter___date] }
                limit: 1000
            ) {
                edges {
                    node {
                        frontmatter {
                            path
                            template <--- add this
                        }
                        ...
```

And lastly we will change the `createPage` call to pull the template dynaically from the fron matter data as follows.
```
        result.data.allMarkdownRemark.edges.forEach(({ node }) => {
            console.log(`Building page ${node.frontmatter.path}...`)
            createPage({
                path: node.frontmatter.path,
                component: templates[node.frontmatter.template],    <--- modify this line
                context: {}, // additional data can be passed via context
            })
        })
```

Now restart the development server and test your http://localhost:8000/contact page. It should now show the contact template.

### Linking Pages

