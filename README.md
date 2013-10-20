# Upgrading To Rails 4

An e-book covering the deprecations, new features and common upgrading
scenarios facing developers upgrading from Rails 3 to Rails 4.

Originally written by [Andy Lindeman](http://andylindeman.com) and now released
under [CC BY 3.0](http://creativecommons.org/licenses/by/3.0/).

Where should you go from here?
* Buy the PDF, ePub, and mobi versions of book at
  <http://upgradingtorails4.com>. **Proceeds since the book was open sourced
  will be donated to <http://cfy.org/>.** You'll receive periodic updates
  as the content is updated in this repository.
* [Build the book yourself](#build-toolchain).
* Contribute to the book. [Pull requests
  accepted](https://github.com/alindeman/upgradingtorails4/pulls).

## Build Toolchain

[kitabu](https://github.com/fnando/kitabu) can export ePub and HTML by default.

```bash
$ bundle install
$ bundle exec kitabu export
$ ls output/
```

As you're hacking on the book, you can have it generate automatically:

```bash
$ bundle exec guard
```

### PDF

PrinceXML is required to build the PDF version of the book. A trial version is
available at [princexml.com](http://www.princexml.com/download/).

### mobi

KindleGen is required to build the mobi version of the book. KindleGen can be
downloaded [from
Amazon](http://www.amazon.com/gp/feature.html?docId=1000765211).
