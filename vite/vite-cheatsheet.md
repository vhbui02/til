# Vite Cheatsheet

## Configuration Vite

`vite.config.[js | mjs | ts]`

```js
import { defineConfig, loadEnv } from "vite";

/** @type {import('vite').UserConfig} */
export default {
  //
};

// or below

// Native ES module syntax (i.e. "type": "module" in "package.json") are supported in config file
// Despite using CommonJS for project
export default defineConfig( /* async */({ command, mode, isSsrBuild, isPreview }) => {
  // async operations (DB, file, network, ...) support

  // ======================================================================== //
  // COMMAND                                                                  //
  // ======================================================================== //
  // $ vite
  // $ vite dev
  // $ vite serve
  if (command === "serve") {
    return {
      // dev specific config
    };
  }
  // $ vite build
  else if (command === "build") {
    return {
      // build specific config
    };
  } else {
    throw `Error: invalid command '${command}'`;
  }

  // ======================================================================== //
  // ENV                                                                      //
  // ======================================================================== //

  // only env var in the current OS environment are available as `process.env.*`
  // CAUTION: Vite defers loading any .env* files AFTER the config has been resolved
  // reason: the loading of some files depend on settings such as `root`, `envDir` and final `mode`

  // => variables in .env, .env.local, .env.[mode] or .env.[mode].local are NOT auto injected into `process.env.*` white `vite.config.*` is running.
  console.log(process.env.VALUE_INSIDE_ENV_FILE); // undefined

  // => They are loaded AFTER the config, and exposed to the application code via `import.meta.env.*` (CAUTION: The variable name is stripped of its `VITE_` prefix)
  console.log("The following examples are run AFTER the config is loaded")
  console.log(import.meta.env.MODE); // 'development' or 'production'
  console.log(import.meta.env.BASE_URL); // derived from `base` config option inside the return object of this function
  console.log(import.meta.env.PROD); // true when running dev server with NODE_ENV='production' and running an app built with NODE_ENV='production', false if otherwise
  console.log(import.meta.env.DEV); // always opposite of PROD
  console.log(!import.meta.env.PROD);
  console.log(import.meta.env.SSR); // whether the app is running in the server

  // CAUTION: security practice is to put `VITE_` prefix to variables exposing to Vite-processed code. DO NOT add `VITE_` prefix to API key, database password, ... or better, such as AWS Secret Manager Parameter Store

  // e.g. .env content
  //
  // VITE_SOME_KEY=123
  // MYSQL_PASSWORD=this-is-not-a-password
  //
  console.log(import.meta.env.VITE_SOME_KEY); // 123
  console.log(import.meta.env.DB_PASSWORD); // undefined

  // ======================================================================== //
  // MODE                                                                     //
  // ======================================================================== //

  console.log(mode);
  // 'development' // $ vite dev      // load .env.development + .env.development.local
  // 'production'  // $ vite build    // load .env.production and .env.production.local
  // 'staging'     // $ vite build --mode staging  // load .env.staging and .env.staging.local

  // The loading order of the environment files:
  //
  // First.   .env              // Loaded in all cases. VCS okay
  // Then.    .env.local        // Loaded in all caces. VCS NOT OKAY
  // Then.    .env.[mode]       // Loaded depends on 'mode'. VCS okay
  // Finally. .env.[mode].local // Loaded depends on 'mode'. VCS NOT OKAY

  // TIPS: DO NOT USE NODE_ENV in Vite, if you're smart enough to know it's a black-zone for poor designing practice.
  // TIPS: IF YOU'RE NOT USING EXPRESSJS, THAT'S EVEN BETTER.

  // ======================================================================== //
  // LOADING ENV FILE DURING CONFIG FILE PROCESSING                           //
  // ======================================================================== //

  // Docs: https://vite.dev/guide/api-javascript.html#loadenv
  const env = loadEnv(mode, process.cwd() /*, prefixes: 'VITE_' */);

  // CAUTION:  3rd param = "" => load all env vars, regardless of its name having `VITE_` or not => bad security practice
  // DO NOT CALL THIS API loadEnv() unless the env file is used DURING the config file processing

  // EXTREMELY CAUTION: env file loaded into import.meta.env.* using 3rd party module and not from Vite's ecosystem => VITE_ prefix is preserved.
  // EXTERMELY CAUTION: loadEnv() stripped off the `VITE_` prefix.

  // Q: when should you use loadEnv() ?
  // A:
  // - config depend on these env (e.g. server port)
  // - they are heavily influenced by `mode`, `command`, ... value, they need to be computed. Best practice is to name it different from the base env var name then set it in `define:`

  return {
    server: {
      // Remember: env.* or import.meta.env.* are all String
      port: env.APP_PORT ? Number(env.APP_PORT) : 8080
    },
    define: {
      // computed env var
      INSERT_ENV_NAME_HERE: JSON.stringify(env.INSERT_ENV_NAME_HERE),
    },
  };
});
```

It's best to maintain ONE `vite.config.*` file, so you can load it implicitly without needing the option `--config`.

By default, `--configLoader bundle` is used, it generated a temporary config file into `node_modules/.vite-temp` directory. Cons: not support in read-only environment such as `docker run --read-only`, therefore not recommenended for security-first production environment.

To tackle the problem, use `vite --configLoader runner` that use module runner which will not create any temp file and transform files on the fly. **NOTE:** it does not support Common JS syntax in config, so be sure to use Native ESModule (ESM) syntax. => Recommend

If you have TypeScript installed, or just plain JavaScript, use `--configLoader native` is a choice to use environment's native runtime, but **updates to the modules imported by the config files aren't detected, therefore prevent auto-restarting Vite server** => Still not recommend

## References

- [Vite Official Documentation's "Configure Vite"](https://vite.dev/config/)
- [Vite Official Documentation's "Env Variables and Modes"](https://vite.dev/guide/env-and-mode.html)
