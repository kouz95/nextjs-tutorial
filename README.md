# Next.js

## Pre-rendering and Data Fetching

Next.js has two forms of pre-rendering

- Static Generation
    - generates the HTML at **build time**. The pre-rendered HTML is then reused on each request.
- Server-side Rendering
    - generates the HTML on **each request**.

The difference is in **when** it generates the HTML for a page.

Static Generation with Data using `getStaticProps`

- `getStaticProps` runs at build time in production, and…
- Inside the function, you can fetch external data and **send it as props** to the page.

To use Server-side Rendering, you need to export `getServerSideProps` instead of `getStaticProps`
from your page. Because `getServerSideProps` is called at request time, its parameter `(context)`
contains request specific parameters.

> You should use `getServerSideProps` only if you need to pre-render a page whose data must be fetched at request time. Time to first byte (TTFB) will be slower than `getStaticProps` because the server must compute the result on every request, and the result cannot be cached by a CDN without extra configuration.

### Client-side Rendering

If you do not need to pre-render the data, you can also use the following strategy (called
Client-side Rendering)

- Statically generate (pre-render) parts of the page that do not require external data.
- When the page loads, fetch external data from the client using JavaScript and populate the
  remaining parts.

> This approach works well for user dashboard pages, for example. Because a dashboard is a private, user-specific page, SEO is not relevant, and the page doesn’t need to be pre-rendered. The data is frequently updated, which requires request-time data fetching.

### SWR

```javascript
import useSWR from 'swr'

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher)
  if (error) return <div>failed to load</div>
  if (!data) return <div>loading...</div>
  return <div>hello {data.name}!</div>
}
```

In this example, the `useSWR` hook accepts a key string and a `fetcher` function. `key` is a unique
identifier of the data (normally the API URL) and will be passed to `fetcher`. `fetcher` can be any
asynchronous function which returns the data, you can use the native fetch or tools like Axios.

The hook returns 2 values: data and error, based on the status of the request.

## Dynamic Routes

### Page Path Depends on External Data

_the case where each page path depends on external data._

- `getStaticPaths` which returns an array of possible values for **id**
- `getStaticProps` which fetches necessary data for the post with **id**

### getStaticPaths (Static Generation)

```javascript
export async function getStaticPaths() {
  return {
    paths: [
      { params: { id: '1' } },
      { params: { id: '2' } }
    ],
    fallback: true // or false 
  }
}
```

```javascript
// catch params of paths,
export async function getStaticProps({ params }) {
  // Fetch necessary data for the blog post using params.id
}
```

```javascript
export async function getStaticPaths() {
  return [
    {
      params: {
        // Statically Generates /posts/a/b/c
        id: ['a', 'b', 'c']
      }
    }
    //...
  ]
}
```

```javascript
export async function getStaticProps({ params }) {
  // params.id will be like ['a', 'b', 'c']
}
```