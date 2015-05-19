### Contributing

To run the site locally, ensure you've bundler installed:

```
sudo gem install bundler
```

Now install jekyll:

```
bundler install  # This reads the repo's Gemfile
```

The site can be run (and automatically regenerated) using:

```
jekyll serve  # Runs at http://localhost:4000/style-guide/ by default
```

Stylistic & semantic changes to the documents themselves require no entries in the change log. Entries in the change log should be reserved for document updates that will influence an engineer's work. E.g: updating the navigation of this style guide requires no entries in the change log, but adding comments about how to add doc strings does.

### Content

is typically found in the root of the repo and are markdown files; i.e. `javascript.md` Pull requests welcome.
