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

```json {filename="package.json"}
{}
```

```js {filename="server.js"}
require("http").createServer((req, res) => {
  res.end("hello world from test-2\n");
}).listen(process.env.PORT || 5000);
```

No framework, no dependencies. The Node buildpack sees `package.json`, and with no `start` script, `npm start` falls back to `node server.js`.

## Shipping It

```sh
dokku apps:create test-2
dokku domains:set test-2 test-2.apps.foo.tools
git push dokku@lab2:test-2 main
dokku letsencrypt:enable test-2
```

About 25 seconds later, `https://test-2.apps.foo.tools` is live with its own Let's Encrypt cert. When I'm done with it:

```sh
dokku apps:destroy test-2 --force
```

## Adding Persistence

A hello world is fun, but a lot of my little ideas need to remember *something*. For that, SQLite is perfect: no database server, just a file.

The catch is that a Dokku container's filesystem gets thrown away on every deploy. The database file has to live on the host and get mounted into the container:

```sh
dokku storage:ensure-directory test-3
dokku storage:mount test-3 /var/lib/dokku/data/storage/test-3:/data
```

The app reads and writes `/data/app.db`, which is really `/var/lib/dokku/data/storage/test-3/app.db` on the host. That's also a handy file to back up.

I used [Bun](https://bun.sh) for this one because SQLite is built in, so there's still nothing to install. Here's `server.ts`, a simple hit counter:

```ts {filename="server.ts"}
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

```dockerfile {filename="Dockerfile"}
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

In a world of complicated dev setups and toolchains, this is a nice. Almost as easy as [FTPing a PHP file to an Apache server](/2021/03/04/20-years-ago-songmeanings/).
