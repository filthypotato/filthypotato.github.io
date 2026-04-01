---
title: "Self-Hosting Immich on Unraid (My Google Photos Replacement)"
date: 2026-04-01
categories: [Homelab, Self-Hosting]
tags: [Immich, Unraid, Docker, Homelab, Privacy, Self-Hosted]
image:
  path: /assets/img/immich-banner.webp
  alt: Immich self hosted photo server
---

> Note: This setup was originally built on Unraid, I’ve since migrated my homelab to Proxmox, but the concepts still apply.

I wanted to move away from Google Photos and start hosting my own photo backup system.

After some trial and error, I deployed **Immich** on my **Unraid homelab** using Docker along with PostgreSQL and Redis.

The result is essentially **my own private Google Photos replacement** running locally on my server.

---

## Why I Wanted to Leave Google Photos

Google Photos is convenient, but it comes with tradeoffs:

- Storage limits
- Subscription fees
- Privacy concerns
- Vendor lock-in

I wanted something that was:

- **Self-hosted**
- **Private**
- **Fast**
- **Works with iPhone auto backups**

Immich checked all those boxes.

---

## My Homelab Environment

The stack running this looks like:

- **Unraid Server**
- Docker containers
- Immich
- PostgreSQL database
- Redis
- 1TB photo storage disk (this turned out to not be enough as I filled the 1TB before they all uploaded. Need more storage, that's a later upgrade)

The containers I deployed were:

```
immich
postgresql15
redis
```

---

## Setting Up PostgreSQL

Immich requires PostgreSQL with **vector support** for machine learning features.

The standard PostgreSQL container does **not** include the required extension, so I switched to the Immich-supported image:

```
ghcr.io/immich-app/postgres:15-vectorchord0.4.3-pgvectors0.3.0
```

Environment variables used:

```
POSTGRES_USER=potato
POSTGRES_DB=immich
POSTGRES_PASSWORD=*****
```

Database storage path:

```
/mnt/cache/appdata/postgresql15
```

---

## Setting Up Redis

Immich also requires Redis for caching and background jobs.

I deployed the official Redis container with the default port:

```
6379
```

No password was required since this runs only inside my internal Docker network.

---

## Deploying the Immich Container

The Immich container connects to both PostgreSQL and Redis.

Important environment variables:

```
DB_HOSTNAME=172.17.0.4
DB_USERNAME=potato
DB_PASSWORD=*****
DB_DATABASE_NAME=immich
DB_PORT=5432

REDIS_HOSTNAME=172.17.0.5
REDIS_PORT=6379

SERVER_HOST=0.0.0.0
SERVER_PORT=8080
```

Web interface port mapping:

```
Host: 8088
Container: 8080
```

Photo storage location:

```
/mnt/disks/New_Volume/photos
```

External library folder:

```
/mnt/disks/New_Volume/libraries
```

Once the container started successfully, the web UI became available at:

```
http://192.168.8.159:8088
```

---

## Creating User Accounts

The first account created in Immich becomes the **admin user**.

After logging in, additional users can be created under:

```
Administration > Users > Create User
```

This allowed me to create separate accounts for:

- Myself
- My wife

Each user gets their own photo library while still allowing **shared albums**.

---

## Mobile Backup (iPhone / Android)

Immich provides a mobile app that automatically backs up photos.

Steps:

1. Install the **Immich app**
2. Enter the server URL

```
http://192.168.8.159:8088
```

3. Login
4. Enable **Auto Upload**

Now every photo taken on our phones automatically backs up to the server.

---

## Migrating Photos from Google Photos

To migrate our existing photos, I used **Google Takeout**.

Steps:

1. Export Google Photos using Takeout
2. Download the archive
3. Extract the files
4. Move them into the Immich storage folder

```
/mnt/disks/New_Volume/photos
```

Then create an external library in Immich:

```
Administration > External Libraries
```

Run a scan and Immich indexes the entire collection.

---

## The Result

Now my homelab provides:

- Private photo backup
- Mobile auto uploads
- Facial recognition
- Fast search
- Shared albums

All without paying for cloud storage.

And most importantly:

**My photos stay in my lab.**

---

## Final Thoughts

Immich has become one of the most impressive self-hosted applications I've deployed.

It genuinely feels like a **self-hosted Google Photos replacement**.

Running it locally gives me:

- full ownership of my data
- no subscription costs
- fast local access

For anyone building a homelab, this is absolutely worth deploying!

---

*If it's not broken, fix it til it is.*
