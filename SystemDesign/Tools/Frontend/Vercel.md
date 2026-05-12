# https://vercel.com/docs/incremental-static-regeneration

With ISR, Vercel knows a path is cacheable before the first request arrives. That's what enables request collapsing, durable storage, 300ms global purges, instant rollbacks, and path grouping. With Cache-Control headers alone, Vercel doesn't know a path is cacheable until it receives the response.