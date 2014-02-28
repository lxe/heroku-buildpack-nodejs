Heroku buildpack: Node.js
=========================

This is the official [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for Node.js apps. If you fork this repository, please **update this README** to explain what your fork does and why it's special.

Cairo Branch
============

This is mojodna's branch of bloomtime's branch of [Heroku's official node.js
buildpack](https://github.com/heroku/heroku-buildpack-nodejs).

We've added `cairo`, `pixman`, and `freetype` to the `vendor` folder so you can
run [`node-canvas`](https://github.com/LearnBoost/node-canvas) on
[Heroku](http://heroku.com/).

Simply do `heroku config:add
BUILDPACK_URL=git://github.com/mojodna/heroku-buildpack-nodejs.git#cairo` and
push as normal.

[vulcan issue 20](https://github.com/heroku/vulcan/issues/20) contains
information about how to build the binaries (more or less). Cairo was built by
hand by using `heroku run bash`, as I couldn't get `vulcan` to correctly use the
`pixman` and `freetype` dependencies when building.  `pixman` and `freetype`
were manually placed in `/app/vendor` and added to `CPATH`, `CPPATH`, and
`LIBRARY_PATH`.

See [Heroku's
Devcenter](https://devcenter.heroku.com/articles/buildpack-binaries) for how to
create your own custom buildpack.

Credits: @bloomtime, @mojodna

How it Works
------------

Here's an overview of what this buildpack does:

- Uses the [semver.io](https://semver.io) webservice to find the latest version of node that satisfies the [engines.node semver range](https://npmjs.org/doc/json.html#engines) in your package.json.
- Allows any recent version of node to be used, including [pre-release versions](https://semver.io/node.json).
- Uses an [S3 caching proxy](https://github.com/heroku/s3pository#readme) of nodejs.org for faster downloads of the node binary.
- Discourages use of dangerous semver ranges like `*` and `>0.10`.
- Uses the version of `npm` that comes bundled with `node`.
- Puts `node` and `npm` on the `PATH` so they can be executed with [heroku run](https://devcenter.heroku.com/articles/one-off-dynos#an-example-one-off-dyno).
- Caches the `node_modules` directory across builds for fast deploys.
- Doesn't use the cache if `node_modules` is checked into version control.
- Runs `npm rebuild` if `node_modules` is checked into version control.
- Always runs `npm install` to ensure [npm script hooks](https://npmjs.org/doc/misc/npm-scripts.html) are executed.
- Always runs `npm prune` after restoring cached modules to ensure cleanup of unused dependencies.

For more technical details, see the [heavily-commented compile script](https://github.com/heroku/heroku-buildpack-nodejs/blob/master/bin/compile).


Documentation
-------------

For more information about using Node.js and buildpacks on Heroku, see these Dev Center articles:

- [Heroku Node.js Support](https://devcenter.heroku.com/articles/nodejs-support)
- [Getting Started with Node.js on Heroku](https://devcenter.heroku.com/articles/nodejs)
- [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
- [Buildpack API](https://devcenter.heroku.com/articles/buildpack-api)


Legacy Compatibility
--------------------

For most Node.js apps this buildpack should work just fine. If, however, you're unable to deploy using this new version of the buildpack, you can get your app working again by using the legacy branch:

```
heroku config:set BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-nodejs#legacy -a my-app
git commit -am "empty" --allow-empty # force a git commit
git push heroku master
```

Then please open a support ticket at [help.heroku.com](https://help.heroku.com/) so we can diagnose and get your app running on the default buildpack.

Hacking
-------

To make changes to this buildpack, fork it on Github. Push up changes to your fork, then create a new Heroku app to test it, or configure an existing app to use your buildpack:

```
# Create a new Heroku app that uses your buildpack
heroku create --buildpack <your-github-url>

# Configure an existing Heroku app to use your buildpack
heroku config:set BUILDPACK_URL=<your-github-url>

# You can also use a git branch!
heroku config:set BUILDPACK_URL=<your-github-url>#your-branch
```

For more detailed information about testing buildpacks, see [CONTRIBUTING.md](CONTRIBUTING.md)


Testing
-------

[Anvil](https://github.com/ddollar/anvil) is a generic build server for Heroku.

```
gem install anvil-cli
```

The [heroku-anvil CLI plugin](https://github.com/ddollar/heroku-anvil) is a wrapper for anvil.

```
heroku plugins:install https://github.com/ddollar/heroku-anvil
```

The [ddollar/test](https://github.com/ddollar/buildpack-test) buildpack runs `bin/test` on your app/buildpack.

```
heroku build -b ddollar/test # -b can also point to a local directory
```

For more info on testing, see [Best Practices for Testing Buildpacks](https://discussion.heroku.com/t/best-practices-for-testing-buildpacks/294) on the Heroku discussion forum.
