# gatsby-source-personio-xml

Source plugin for pulling data into Gatsby from a Personio XML feed.

## Install

```bash
npm install --save gatsby-source-personio-xml
```

or

```bash
yarn add gatsby-source-personio-xml
```

## How to use

In `gatsby-config.js`:

```js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-source-personio-xml`,
      options: {
        url: `https://{username}.jobs.personio.de/xml`,
      },
    },
  ],
};
```

## How to query

You may access the following data node types:

| Node                 | Description                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `PersonioPosition`   | The job postings                                                                                                       |
| `PersonioDepartment` | The departments from `department` field in the [XML](https://developer.personio.de/docs/retrieving-open-job-positions) |
| `PersonioOffice`     | The offices from `office` field in the [XML](https://developer.personio.de/docs/retrieving-open-job-positions)         |

The field names follow the scheme in the [Personio XML feed](https://developer.personio.de/docs/retrieving-open-job-positions).

To retrieve a list of all departments with their job postings the following GraphQL query should work:

```graphql
allPersonioDepartment {
  edges {
    node {
      id
      name
      positions {
        id
        positionId
        recruitingCategory
        office {
          id
          name
        }
        employmentType
        schedule
        seniority
        subcompany
        yearsOfExperience
        name
        jobDescriptions {
          name
          value
        }
      }
    }
  }
}
```


## Adding custom fields

Personio allows you to define custom fields which are not mapped automatically. For these occasions you 
can customize the XML parsing and GraphQL mapping via the following configuration options: 

```js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-source-personio-xml`,
      options: {
        url: `https://{username}.jobs.personio.de/xml`,
        cusomizeXmlMapping: (newNode, xmlNode) => {
          const createdAt = select("string(createdAt)", xmlNode)
          return { ...newNode, createdAt, keywords }
        },
        customizeNodeMapping: (gatsbyNode, originalMappedItem) => {
          return {
            ...gatsbyNode,
            createdAt: originalMappedItem.createdAt,
          }
        },
      },
    },
  ],
};
```

In addition you will want to define your own Graphql Schema Mapping in your own `gatsby-node.js`:

```js
exports.createSchemaCustomization = ({ actions }) => {
  const { createTypes } = actions

  createTypes(`
    type PersonioPosition implements Node {
      createdAt: Date
    }
    `)
```