Nella sezione Upgrading (https://chirpy.cotes.page/posts/getting-started/#upgrading) seguire le linee guida dell'opzione 1.

## I suggest to get a BACKUP copy before upgrading.

OPZIONE 1: 

If you are using the theme gem (there will be gem "jekyll-theme-chirpy" in the Gemfile), editing the Gemfile and update the version number of the theme gem, for example:
```diff
- gem "jekyll-theme-chirpy", "~> 3.2", ">= 3.2.1"
+ gem "jekyll-theme-chirpy", "~> 3.3", ">= 3.3.0"
```
And then execute the following command:

```console
  $ bundle update jekyll-theme-chirpy
 ```
As the version upgrades, the critical files (for details, see the Startup Template) and configuration options will change. Please refer to the Upgrade Guide to keep your repo’s files in sync with the latest version of the theme.

The critical files are:
````console
.
├── _config.yml
├── _data
├── _plugins
├── _tabs
└── index.html
````

