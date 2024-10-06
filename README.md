# Bug demontration repository

This repo reproduce the query ambiguity occurring after the upgrade from Strapi v4 to v5.

## The issue

During the first run of Strapi after its update to v5, the DB migration 5.0.0-02-document-id can fail.

It happens when a content type has those 2 characteristics:
1) Internationalization is turn on
2) One of its field is named as [name_of_the_type]Id

## Log of the error

```
┌───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                                                                           │
│   MigrationError: Migration 5.0.0-02-created-document-id (up) failed: Original error:                                                     │
│       SELECT `tests`.id as id, group_concat(DISTINCT `inv_test_id`) as other_ids                                                          │
│       FROM `tests`                                                                                                                        │
│       LEFT JOIN `tests_localizations_links` ON `tests`.id = `tests_localizations_links`.`test_id`                                         │
│       WHERE document_id IS NULL                                                                                                           │
│       GROUP BY `test_id`                                                                                                                  │
│       LIMIT 1;                                                                                                                            │
│        - ambiguous column name: test_id                                                                                                   │
│       at /home/mottet/git/strapi-failing-v5-db-migration/node_modules/umzug/lib/umzug.js:169:27                                           │
│       at async Umzug.runCommand (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/umzug/lib/umzug.js:125:20)                  │
│       at async Object.up (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/database/dist/index.js:6531:7)             │
│       at async Object.up (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/database/dist/index.js:6555:11)            │
│       at async Object.sync (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/database/dist/index.js:2246:9)           │
│       at async Strapi.bootstrap (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/core/dist/Strapi.js:351:5)          │
│       at async Strapi.load (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/core/dist/Strapi.js:315:5)               │
│       at async Module.develop (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/strapi/dist/node/develop.js:177:28)   │
│       at async action (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/@strapi/strapi/dist/cli/commands/develop.js:18:5)     │
│       at async Command.parseAsync (/home/mottet/git/strapi-failing-v5-db-migration/node_modules/commander/lib/command.js:923:5)           │
│                                                                                                                                           │
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Why

It is because the GROUP BY in the SQL request does not precise the column's table which open up to possible ambiguity.

## Reproduction

In this repository, a sqlite db was push with a content type named Test in it that contains a field testId. This reproduce the conflict explained above.

1) Run `yarn install`
2) Run `yarn develop`

## The fix

Add the column's table in the GROUP BY of the query.

## Login info

**Login:** test@test.com

**Password:** P@ssw0rd
