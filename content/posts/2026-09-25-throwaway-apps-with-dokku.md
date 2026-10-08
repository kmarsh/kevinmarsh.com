---
title: "Throwaway Apps with Dokku and a Wildcard Domain"
date: 2026-09-25
slug: throwaway-apps-with-dokku
draft: false
ai: true
---

I have a [Dokku](https://dokku.com) box in my lab that runs a pile of real things: side projects, a couple of databases, and the backups for all of it. It's my own little Heroku, and it's been great.

What it wasn't great at was the *tiny* stuff. Sometimes I want a single endpoint: a webhook receiver, a JSON blob for a shortcut on my phone, or a quick "does this idea even work" page. Spinning up a new app on a new domain with DNS and a cert felt like too much ceremony for something I might delete tomorrow. So those ideas mostly didn't happen.

The fix turned out to be one DNS record:

## The Wildcard

I pointed `*.apps.foo.tools` at the Dokku server now any name under `apps.foo.tools` resolves to the box, and Dokku's nginx routes each request by hostname. A wildcard only matches names that don't have their own record, so none of the other domains on the server notice.

## The App

This is the entire app, all two files:

```json
{}
```

That's `package.json`. And `server.js`:

```js
require("http").createServer((req, res) => {
  res.end("hello world from test-2\n");
}).listen(process.env.PORT || 5000);
```

No framework, no dependencies. The Node buildpack sees `package.json`, and with no `start` script, `npm start` falls back to `node server.js`.

## Shipping It

```sh
ssh lab2 'dokku apps:create test-2 && dokku domains:set test-2 test-2.apps.foo.tools'
git push dokku@lab2:test-2 main
ssh lab2 'dokku letsencrypt:enable test-2'
```

About 25 seconds later, `https://test-2.apps.foo.tools` is live with its own Let's Encrypt cert. When I'm done with it:

```sh
ssh lab2 'dokku apps:destroy test-2 --force'
```

## Buildpack or Dockerfile?

I tried it both ways. The Dockerfile version fits in a *single* file by inlining the JavaScript with a heredoc `COPY`:

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine
COPY <<'EOF' /app/server.js
require("http").createServer((req, res) => {
  res.end("hello world from test-1\n");
}).listen(process.env.PORT || 5000);
EOF
EXPOSE 5000
CMD ["node", "/app/server.js"]
```

Here's how they compared on the same hello world:

| | Dockerfile (alpine) | Buildpack (herokuish) |
|---|---|---|
| Files | 1 | 2 |
| Image size | 167 MB | 1.27 GB |
| Memory | ~10 MiB | ~54 MiB |
| Extra setup | `ports:set http:80:5000` | none |

The buildpack image looks huge, but most of it is a base layer every buildpack app on the server shares, so each new app costs much less disk than that. The memory gap is mostly `npm start` hanging around as a parent process. A one-line `Procfile` with `web: node server.js` should close most of it.

## Adding a Database

A hello world is fun, but a lot of my little ideas need to remember *something*. For that, SQLite is perfect: no database server, just a file.

The catch is that a Dokku container's filesystem gets thrown away on every deploy. The database file has to live on the host and get mounted into the container:

```sh
dokku storage:ensure-directory test-3
dokku storage:mount test-3 /var/lib/dokku/data/storage/test-3:/data
```

The app reads and writes `/data/app.db`, which is really `/var/lib/dokku/data/storage/test-3/app.db` on the host. That's also a handy file to back up.

I used [Bun](https://bun.sh) for this one because SQLite is built in, so there's still nothing to install. Here's `server.ts`, a hit counter:

```ts
import { Database } from "bun:sqlite";

const db = new Database("/data/app.db", { create: true });
db.run("CREATE TABLE IF NOT EXISTS hits (at TEXT NOT NULL)");

Bun.serve({
  port: process.env.PORT || 5000,
  fetch() {
    db.run("INSERT INTO hits VALUES (datetime('now'))");
    const { n } = db.query("SELECT count(*) AS n FROM hits").get() as { n: number };
    return new Response(`hello from test-3, visit #${n}\n`);
  },
});
```

And the `Dockerfile`:

```dockerfile
FROM oven/bun:1-alpine
COPY server.ts /app/
CMD ["bun", "/app/server.ts"]
```

Leaving out `EXPOSE` lets Dokku fall back to mapping port 80 to 5000, so there's no `ports:set` step this time. After a full `dokku ps:rebuild`, the count picks up right where it left off. The whole thing is an 87 MB image idling at under 4 MiB of memory, which is the smallest of the bunch.

## One Gotcha

With a wildcard in place, a request for a name that *isn't* an app, like `typo.apps.foo.tools`, still reaches the server. Without a catch-all server block, nginx hands it to whatever it considers the default site. On my box that meant `nope.apps.foo.tools` happily served one of my real apps, cert and all.

These are all internal toys for me, so I've left it alone. If that matters to you, Dokku's docs recommend a default vhost that drops unknown hosts:

```nginx
# /etc/nginx/conf.d/00-default-vhost.conf
server { listen 80 default_server; server_name _; return 444; }
server { listen 443 ssl default_server; server_name _; ssl_reject_handshake on; return 444; }
```
