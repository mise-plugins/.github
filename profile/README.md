# mise plugins

Making software development safer, faster, and simpler.

# How it works

[mise](https://mise.jdx.dev) is built with its predecessor [asdf](https://asdf-vm.com)'s plugin model. asdf/mise plugins are how each language/tool work and are git repos with a set of bash script. For example, tuist can be installed by either of the following:

```sh
mise use --global tuist@latest
```

```sh
asdf plugin add tuist
asdf install tuist latest
asdf global tuist latest
```

Both asdf/mise know that in order to install tuist, it first needs [asdf-tuist](https://github.com/mise-plugins/asdf-tuist). In asdf, that mapping of "short name" (tuist in this case) to repo url (https://github.com/mise-plugins/asdf-tuist) is managed in the [asdf-vm/asdf-plugins](https://github.com/asdf-vm/asdf-plugins) repo. mise[^fetch] uses a clone of this repo ([mise-plugins/registry](https://github.com/mise-plugins/registry)).

Of course plugins can also be fetched by specifying the full repo url:

```
mise plugins install https://github.com/mise-plugins/asdf-tuist
```

However this (unsurprisingly) is not commonly done—especially in mise where it's possible to perform simple commands like `mise install` to install all plugins/tools in the .mise.toml/.tool-version file. Users like the simplicity of not needing to look this information up ahead of time.

## Problems with [asdf-vm/asdf-plugins](https://github.com/asdf-vm/asdf-plugins)

This model has supply chain risks and quality issues. There is little to no vetting that happens before a "shortname" is added to asdf's registry. The asdf maintainers have no control over future development of that plugin. If the plugin author is a bad actor or compromised, users of the plugin become exposed
either when installing the plugin or updating it to the latest version.

Also, because plugins are not hard to build, they are often built by individuals to solve their own problems, submitted to the registry, then not maintained.
Often PRs sit with unresponsive maintainers and ultimately the plugin may be forked and the registry switched to a new owner. This increases the supply chain
problem by lowering the bar to get access to plugins and also diverges the community. It becomes harder to follow development of a plugin. Users which
previously added a plugin may be using an unmainted one and updating to the latest version will not move them to the maintained one.

## Plugin Development in mise-plugins

In this org, we plan to mitigate these issues by changing the ownership model to vet changes and prevent bad actors from having commit access to plugins.
Only the [owners](#owners) will have commit access to plugins in this org. They will serve largely as reviewers to vet changes. Development will still
need to be performed by community plugin authors via reviewed PRs.

Not only will this improve the supply chain security for mise users, we think this will increase quality of plugins as well. Plugins should not sit around
with bugs for months waiting for unresponsive maintainers to merge.

## Building a new plugin

- Read asdf's [creating plugins guide](https://github.com/asdf-vm/asdf/blob/master/docs/plugins/create.md)
- Consider using asdf's [Template](https://github.com/asdf-vm/asdf-plugin-template) which has the core functionality to tools published to GitHub releases and CI for GitHub/GitLab/CircleCI out of the box.
  - refer to https://github.com/mise-plugins/mise-poetry/tree/main/.github/ for mise-specific workflows 
- Submit a transfer request to @jdx and he will put it into the org.

## Trusted Plugins

mise has the concept of "trusted" and "untrusted" plugins. Trusted plugins will automatically install when running commands like `mise install` or `mise use`,
however untrusted plugins will receive the following warning/confirmation:

```sh-session
$ mise install zigmod@latest
⚠️  zigmod is a community-developed plugin: https://github.com/kachick/asdf-zigmod
 Would you like to install zigmod [y/n]?
```

Trusted plugins may be of one of two types, they can be either first-party or hosted in this org with reviewed changes by trusted owners.

## First-Party Plugin Authors

Those that maintain asdf plugins for tools they also own (e.g.: Hashicorp maintains their own asdf/mise plugins) can mark their plugin as first-party by
adding the following to the [registry](https://github.com/mise-plugins/registry) in a PR:

```toml
first-party = true
```

## Plugin Authors (not first-party authors)

We encourage plugin authors to submit their plugins to this repo to increase the security for users of the plugin and to avoid the warning message
seen when first installing a non-official plugin.

If you'd like to submit your plugin, you can either [submit a discussion](https://github.com/orgs/mise-plugins/discussions/new?category=transfer-request) to
start a conversation or simply request to transfer the repo to @jdx on GitHub and he'll know what to do.

Optionally, we can put a CODEOWNERS file in the root of the plugin with your handle so users know who built/maintains the plugin.

> [!IMPORTANT]
> This will not work like asdf-community where individuals formerly maintained plugins will retain commit access. Once your repository is in
> mise-plugins, you will need to submit a PR and have it reviewed by one of the [owners](#owners) to be merged.

## Owners

In order to keep things secure, this org will only ever have a handful of dedicated owners to review PRs. Here are who they are: 

### [Jeff Dickey (@jdx)](https://github.com/jdx)

Jeff created mise and has been an open source developer for many years. He's worked at many companies including Heroku (Salesforce), Meta, and currently Amazon. He has a deep passion for dev tools especially CLI design.

### [Justin "J.R." Hill (@booniepepper)](https://github.com/booniepepper)

J.R. is a FOSS enthusiast and a code creative. His personal projects can be found at [so.dang.cool](https://so.dang.cool),
and he currently serves as a Staff Engineer at Onward.

> I developed an interest in the software supply chain during the 9 years I spent working at Amazon. There I provided
> company-wide security incident response to CVEs and embargoed zero-day exploits. I also got to help maintain the company's
> shared code bases (used by Amazon, AWS, Twitch, etc) of >100k polyglot software repositories, from which >575k projects
> are built, for applications that range from the web, cloud infrastructure, robotics, aviation, and more. 

### [Pedro Piñera Buendía (@pepicrft)](https:///github.com/pepicrft)

Pedro created [Tuist](https://tuist.io), a tool to scale development for Apple platforms. He's an open-source advocate and loves building developer tools that spark joy when using them. 

## FAQs

### Can asdf users use this repo?

Probably, but we recommend using mise over asdf. While all asdf plugins should work in mise, the inverse is not necessarily the case. mise has features that asdf may never support. We may change our guidance on this if we either make a commitment to a unified plugin syntax or otherwise make this registry consumable in asdf.

That said, if you want to use this anyways knowing some plugins may not work correctly, you can do so in asdf with the following:

```sh
rm -rf ~/.asdf/repository
git clone https://github.com/mise-plugins/registry ~/.asdf/repository
```

[^fetch]: In asdf, this repo is cloned locally to ~/.asdf/repository and updated occasionally by performing a `git pull`.
