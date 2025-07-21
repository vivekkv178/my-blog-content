## Dynamic Open Graph Images in Next.js

Social media previews are important. Whether it's Twitter, LinkedIn, or WhatsApp — having a good-looking **Open Graph (OG) image** can dramatically improve the visibility of your blog posts.

In this post, I’ll share how I:

* Used CDN-hosted OG image urls stored in my database
* Cached blog metadata using React's built-in **`cache`** utility
* Set up dynamic Open Graph tags for each blog post in Next.js

### 📝 Disclaimer  
> **This content was generated with the assistance of AI. Please conduct your own due diligence before applying any information presented here.**

---

### ✅ The Goal

I wanted each blog post to have:

* A unique OG image (`og:title`, `og:image`)
* Metadata rendered dynamically for better SEO and social sharing
* A clean and efficient setup — fetching blog data once and reusing it for both rendering and meta tags

---

### 🔧 Tech Stack

* Next.js App Router
* `cache` utility from React
* CDN-hosted OG images (stored as part of blog metadata in the database)

---

### 🖼️ OG Image Strategy

Instead of generating OG images on-the-fly, I uploaded them to a CDN ahead of time and stored their URLs in my database. Each blog post entry in the database includes a field like:

```json
{
  "title": "React State Management Simplified",
  "author": "Vivek K V",
  "ogImage": "https://cdn.mysite.com/og/react-state-management.png"
}
```

No need to build the image URL from the slug — I simply fetch the `ogImage` field along with the rest of the metadata.

---

### 🧠 Caching Metadata Using `cache`

When a blog post is accessed, I need metadata like title, author, and slug to:

* Render the blog page content
* Generate dynamic metadata for OG tags

To avoid duplicate API calls, I used the `cache` utility from React (available in Next.js App Router).

```ts
// utils/getBlogMetadata.ts
import { cache } from 'react';

export const getBlogMetadata = cache(async (slug: string) => {
  const res = await fetch(`${process.env.API_URL}/blog/${slug}`);
  if (!res.ok) throw new Error('Failed to fetch blog metadata');
  return res.json();
});
```

This ensures that even if `getBlogMetadata(slug)` is called in multiple places, the API call happens **only once per request**.


---

### 🧩 Using Cached Metadata in `generateMetadata()`

Next.js allows us to dynamically generate SEO tags for each route using an async `generateMetadata` function.

```ts
// app/blog/[slug]/page.tsx
import { getBlogMetadata } from '@/utils/getBlogMetadata';

export async function generateMetadata({ params }) {
  const { title, author, ogImage } = await getBlogMetadata(params.slug);

  return {
    title,
    openGraph: {
      title,
      description: `A blog post by ${author}`,
      images: [ogImage],
    },
  };
}
```

---

### 💻 Rendering the Page with the Same Metadata

```tsx
// app/blog/[slug]/page.tsx
import { getBlogMetadata } from '@/utils/getBlogMetadata';

export default async function BlogPostPage({ params }) {
  const { title, author, content, ogImage } = await getBlogMetadata(params.slug);

  return (
    <article className="prose">
      <h1>{title}</h1>
      <p>By {author}</p>
      <img
        src={ogImage}
        alt={`OG image for ${title}`}
      />
    </article>
  );
}
```

---

### ⚡ Benefits

* ✅ **Fast and SEO-optimized previews**
* ✅ **Clean code with centralized metadata fetching**
* ✅ **Easy to maintain and scale**

---

### 🧪 Bonus Tip

You can preview and validate how your OG tags render using [https://www.opengraph.xyz](https://www.opengraph.xyz)

---

### 💬 Final Thoughts

This setup gave me the best of both worlds: dynamic SEO with Open Graph support, and minimal complexity in managing images and metadata. By caching metadata at the right layer and storing OG image URLs in my database, I was able to deliver fast and reliable social previews for every blog post — with clean, modern Next.js architecture.
