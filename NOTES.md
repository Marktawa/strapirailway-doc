## How to self-host Strapi on Railway

### Description

A practical guide to deploying Strapi on Railway with both SQLite and PostgreSQL setups. Covers how to handle persistent storage, optimize costs with sleep mode, and compares the experience to Strapi Cloud. Perfect for developers looking to self-host their Strapi apps and avoid common Railway deployment pitfalls.

### Justification
- Railway has updated their features (Railway Metal)
- There are a few issues in Strapi community asking for specifics with regards to deploying Strapi on Railway
    - https://discord.com/channels/811989166782021633/1388004956404977784/1388004956404977784
    - https://discord.com/channels/811989166782021633/1367853457196847314/1367853457196847314
    - https://discord.com/channels/811989166782021633/1277553801372631143/1277553801372631143
    - https://discord.com/channels/811989166782021633/1334650673916022805/

### Content Objective

To showcase the ways one can host a Strapi app on Railway. Special attention will be made to making storage persist.

### Content Outline

- Introduction
- Prerequisites
- Deploying Strapi on Railway using SQLite db
- Deploying Strapi on Railway using PostgreSQL db
- Making storage persist
- App sleep mode to save costs
<!-- Split deployment just in case of rejection -->
- Comparison with Strapi Cloud
- Conclusion

### Content Brief: 
- Target audience: Members of the Strapi community who self-host Strapi on Railway or are looking for an option on deploying Strapi using a third party.
- Keywords: `strapi railway deployment`, `self-host strapi`, `strapi railway hosting`, `strapi postgresql railway`