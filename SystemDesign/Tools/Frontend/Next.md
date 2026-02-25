# [Nest.js](https://www.youtube.com/watch?v=k7o9R6eaSes)

**Next.js** is a popular open-source web development framework built on top of **React**. While React is a library for building user interfaces, Next.js provides the "architecture" and tools needed to build full-scale, production-ready web applications.

### What is Next.js?

Often called "The React Framework for the Web," Next.js simplifies the development process by handling complex features out of the box that you would otherwise have to configure manually in a standard React app.

- **Rendering Options:** It allows for Server-Side Rendering (SSR), Static Site Generation (SSG), and Incremental Static Regeneration (ISR), which makes websites much faster and better for SEO.
- **File-Based Routing:** You don't need a separate library like React Router; any file added to the `app` or `pages` directory automatically becomes a route.
- **Full-Stack Capabilities:** It allows you to write backend code (API routes) in the same project as your frontend.
- **Optimization:** It automatically optimizes images, fonts, and scripts to improve performance.

---

### What Build Tool Does it Use?

It kinda itself act as a buildtool, Next.js uses a combination of powerful tools to compile, bundle, and optimize your code. As of 2025, the framework is in a transition period between two major technologies:

#### 1. The Compiler: SWC

Next.js uses **SWC** (Speedy Web Compiler) as its primary compiler. SWC is written in **Rust** and is significantly faster than the older industry standard, Babel. It handles:

- **Transpilation:** Turning modern JavaScript/TypeScript into code browsers can understand.
- **Minification:** Shrinking your code size for production.

#### 2. The Bundler: Turbopack & Webpack

The "bundler" is what takes all your individual files and packages them together.

- **Turbopack (The New Standard):** Developed by Vercel (the creators of Next.js), Turbopack is an incremental bundler written in Rust. In the latest versions of Next.js (like version 15 and 16), it is the default for local development because it is up to **700x faster** than Webpack for large projects.
- **Webpack (The Legacy Standard):** For years, Webpack was the default. While Next.js is moving toward Turbopack for everything, Webpack is still used for production builds in many configurations or if you have custom plugins that haven't migrated to the Turbo ecosystem yet.

### Server Action

- **Without `"use server"`:** The function is just private code inside the house. The browser has no way to "call" it.
- **With `"use server"`:** You are telling Next.js to build a hidden bridge (an HTTP POST endpoint). When the user clicks the button, the browser sends a request across that bridge to execute that specific function on the server.

<br />
<br />
<br />

# [Metadata](https://nextjs.org/docs/app/getting-started/metadata-and-og-images)

### **The `generateMetadata` Function in Next.js (App Router)**

The `generateMetadata` function is an asynchronous function used to dynamically generate metadata (like **title**, **description**, and **openGraph** tags) for a specific route segment based on fetched data or route parameters. This is crucial for SEO and web shareability.

- **Server Components Only:** `generateMetadata` can only be exported from **Server Components** (within `layout.js` or `page.js` files).
- **Dynamic Metadata:** Unlike a static `metadata` object, this function runs at request time (or build time for static generation) and can perform data fetching to populate the metadata fields dynamically.
- **Automatic Head Tag Generation:** Next.js automatically resolves the returned data and generates the appropriate `<head>` tags for the page.

- **Parameters:** The function receives route `params` and `searchParams` as arguments, allowing access to dynamic route segments and URL query parameters.
- **Data Fetching:** Next.js and React automatically **memoize** `fetch` requests within `generateMetadata` and the page component itself, so the same data is only fetched once, even if called in multiple places.

```javascript
// generateMetadata function
export const generateMetadata = async ({ params }: { params: { slug: string } }): Promise<Metadata> => {
  const product = await getProduct(params.slug);

  return {
    title: product.name,
    description: product.description,
    openGraph: {
        images: product.thumbnail,
    },
  };
};

// Page component
const Page = async ({ params }: { params: { slug: string } }) => {
  const product = await getProduct(params.slug); // Data fetching is memoized

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
};
```

- **Streaming Metadata:** For dynamic pages, Next.js can stream the UI first and inject the metadata into the `<body>` later, which improves perceived performance. However, for search engine bots, it ensures the metadata is present in the `<head>` tag when the initial HTML is served.
- **Merging:** Metadata defined in nested layouts and pages is automatically merged, with child segments overriding parent fields for the same key.

---

<!-- TODO -->

* https://nextjs.org/docs/app/api-reference/config/next-config-js
* https://nextjs.org/docs/app/api-reference/functions/generate-static-params