# pnpm ignores workspace dependency files property

`pnpm deploy` ignores `files` property in `package.json` and `.npmignore` for workspace dependencies. All package contents are copied. This may include excessive amounts of source code (adding to the total size of a Docker image for example) and **secret files** ignored by VCS. Those files may later be accessed in Docker containers exposing secrets used to compile the code (S3 keys etc).

## Reproduction steps

1) Run `pnpm install`
2) Create `very-secret` file (it is ignored by `.gitignore`) `touch packages/ui-files/very-secret && touch packages/ui-npmignore/very-secret`
3) Create `.env` file in package dirs: `touch packages/ui-files/.env && touch packages/ui-npmignore/.env`
4) Run `pnpm deploy -F app-files app-files-prod --prod` and then `ls -A app-files-prod/node_modules/ui-files app-files-prod/node_modules/ui-npmignore`.

    Should see
    ```
    app-files-prod/node_modules/ui-files:
    .env         dist.js      package.json src.js       very-secret

    app-files-prod/node_modules/ui-npmignore:
    .env         .npmignore   dist.js      package.json src.js       very-secret
    ```

5) Run `pnpm deploy -F app-files app-npmignore-prod --prod` and then `ls -A app-npmignore-prod/node_modules/ui-files app-npmignore-prod/node_modules/ui-npmignore`.


    Should see
    ```
    app-npmignore-prod/node_modules/ui-files:
    .env         dist.js      package.json src.js       very-secret

    app-npmignore-prod/node_modules/ui-npmignore:
    .env         .npmignore   dist.js      package.json src.js       very-secret
    ```

## Expected behaviour

In any of the packages `ls -A` should output only `dist.js      package.json`.
