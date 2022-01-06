# Releasing the Code

The code is manually built and released in GitHub.  I don't release often enough for it to be worth building a more automated process.

1. Set up release information:

```
$ export GTML_VERSION=3.6.0
$ export GTML_DATE=$(date +'%d %b %Y')
```

2. Update the changelog:

```
$ sed -i "s/^Version $GTML_VERSION\s\s*unreleased/Version $GTML_VERSION     $GTML_DATE/g" Changelog
$ git commit -m "Release $GTML_VERSION" Changelog
```

3. Build the tarball

```
$ tar zcvf gtml-$GTML_VERSION.tar.gz gtml Changelog LICENSE CREDITS README.md docs
```

4. [Tag a new release](https://github.com/pronovic/gtml/releases/new) at GitHub, attaching the tarball generated above
