---
layout: post
title: "Web asset versioning in 30 lines, with Makefile"
date: 2015-08-30 14:53:56 +0900
comments: true
published: true
categories: 
---

I've been using `Makefile` to compile web apps recently, and it's been
liberating. I want to show how even the most complex-seeming tasks can be
achieved with terseness and finesse using a `Makefile`.

Here, we'll tackle the issue of _asset versioning_, that is, calculating a
checksum for each file and appending it to it's base name - a powerful, fool
proof way to achieve cache-busting. We're gonna implement the equivalent of
[gulp-rev][gulp-rev] or [grunt-assets-versioning][grunt-assets-versioning] using
pure bash ;)

Let's setup an environment. We'll need a source folder with files and a final
destination folder to work with.

```make
SRC_FOLDER   := src
DIST_FOLDER  := dist
```

Now we need to build a list of files that we want to end up with. For this
example I'm going to assume the `src` folder looks something like this:

```
/src
  index.html
  index.js
  index.css
    /images
      background.png
      background@2x.png
```

Let's use `find` to do this:

```make
SRC_FILES := $(shell find $(SRC_FOLDER) -type f)
# Our destination should look like a one-to-one copy of src
DIST_FILES = $(patsubst $(SRC_FOLDER)/%, $(DIST_FOLDER)/%, $(SRC_FILES))
```

Now that we know our inputs and outputs, we can hook things up. You might have
noticed that our destination file names don't have checksums appended to
them. The way I like to do it is to keep _a copy of the original and revisioned
version_. It helps make with selective re-compilation, and acts as a fallback if
by chance a file reference has not been replaced properly.

Not all files will want to have their names changed. Typically, you don't want
to move your `index.html` file to a different name. So we store a var for all
the files that we do want to move:

```make
# We're saying, only choose html/css/js/png/jpg files, but exclude index.html
FILES_TO_VERSION = $(strip $(shell echo $(SRC_FILES) \
					| grep -e  ".*\.\(html\|css\|js\|png\|jpg\)" \
					| grep -v -e ".*index\.html"))
```

First file we need to build is a manifest file. `gulp-rev` generates a json file
but a space-delimited list should do fine.

```make
# Here we're saying that we need all the files available in order to generate
# this manifest. It's a given in this scenario, but if you are revisioning some intermediate
# build folder this rule basically instructs make to build the intermediate
# folder first.
$(DIST_FOLDER)/assets-manifest: $(FILES_TO_VERSION)
	# Ensures that the directory exists. Good command to have before each rule.
	@mkdir -p $(@D)
	@echo "Creating assets versioning file -> $@"
	# Here we have a load of bash. Pretty much does this:
	# 1. Looking at all the files that we want to version
	# 2. Turn them into lines with the filename and (md5 hash)[0..4]. i.e
	#        src/index.css j23jd
	# 3. Transform this into the original file, and new file name. i.e
	#        src/index.css src/index.j23jd.css
	# 4. Strip the src folder part. i.e
	#        /index.css /index.j23jd.css
	# 5. Write to the assets file
	@echo $(FILES_TO_VERSION)   | xargs -n1 -I% sh -c 'echo %; md5 -q % | cut -c 1-5;' \
								| xargs -n2 \
								| sed 's|\([^ ]*\)\.\([^\.]*\) \(.*\)|\1.\2 \1.\3.\2|' \
								| sed 's|$(SRC_FOLDER)||g' \
								> $@
```

Your manifest file will look something like this:

```sh
$ cat dist/assets-manifest
/index.html /index.3jsl2.html
/index.js /index.93jfk.js
/index.css /index.j23jd.css
/images/background.png /images/background.jdlm1.png
/images/background@2x.png /images/background@2x.19djm.png
```

Once we have the manifest file, using it, we can build each file in `dist`
individually.

```make
# We're saying, every file in dist is dependent on the file of the same
# name in src AND the asset manifest file.
$(DIST_FOLDER)/%: $(SRC_FOLDER)/% $(DIST_FOLDER)/assets-manifest
	@mkdir -p $(@D)
	# Copy the original file to the new location. We'll modify it in-place after
	@echo "Copying $< -> $@"
	@cp $< $@
	# If we have a text file, then replace all asset references using the
	# manifest file. Note here that we are doing 'naive' replacement, it will
	# only work with absolute paths. I don't see a reason to not use absolute paths
	# in your assets, so this probably isn't a real limitation.
	@if file $< | grep -q text; then \
		echo "Updating asset references $< -> $@"; \
		# The following is pretty complex but the end result is in-file \
		# replacement of all file references with their versioned path. \
		cat $(DIST_FOLDER)/assets-manifest | xargs -n2 -I% sh -c 'p=(`echo "%"`); sed -i "" "s|$${p[0]}|$${p[1]}|g" $@'; \
	 fi
	# Now we make another copy of the same file to it's new versioned path, if it
	# is referenced in the manifest. Here you can easily swap `cp` for `mv` if you
	# don't want to keep the original.
	@NEW_BASE_NAME=$$(cat $(DIST_FOLDER)/assets-manifest | grep $* | sed 's|[^ ]* ||'); \
		# Get the new filename and if it exists copy to the new basename \
		if [ -n "$$NEW_BASE_NAME" ]; then \
			echo "Creating versioned asset $@ -> $(DIST_FOLDER)$$NEW_BASE_NAME"; \
			cp $@ $(DIST_FOLDER)$$NEW_BASE_NAME; \
		fi
```

And the **final result** is...

```make
SRC_FOLDER   := src
DIST_FOLDER  := dist

SRC_FILES := $(shell find $(SRC_FOLDER) -type f)
DIST_FILES = $(patsubst $(SRC_FOLDER)/%, $(DIST_FOLDER)/%, $(SRC_FILES))

FILES_TO_VERSION = $(strip $(shell echo $(SRC_FILES) \
					| grep -e  ".*\.\(html\|css\|js\|png\|jpg\)" \
					| grep -v -e ".*index\.html"))

$(DIST_FOLDER)/assets-manifest: $(SRC_FILES)
	@mkdir -p $(@D)
	@echo "Creating assets versioning file -> $@"
	@echo $(FILES_TO_VERSION)   | xargs -n1 -I% sh -c 'echo %; md5 -q % | cut -c 1-5;' \
								| xargs -n2 \
								| sed 's|\([^ ]*\)\.\([^\.]*\) \(.*\)|\1.\2 \1.\3.\2|' \
								| sed 's|$(SRC_FOLDER)||g' \
								> $@

$(DIST_FOLDER)/%: $(SRC_FOLDER)/% $(DIST_FOLDER)/assets-manifest
	@mkdir -p $(@D)
	@echo "Copying $< -> $@"
	@cp $< $@
	@if file $< | grep -q text; then \
		echo "Updating asset references $< -> $@"; \
		cat $(DIST_FOLDER)/assets-manifest | xargs -n2 -I% sh -c 'p=(`echo "%"`); sed -i "" "s|$${p[0]}|$${p[1]}|g" $@'; \
	 fi
	@NEW_BASE_NAME=$$(cat $(DIST_FOLDER)/assets-manifest | grep $* | sed 's|[^ ]* ||'); \
		if [ -n "$$NEW_BASE_NAME" ]; then \
			echo "Creating versioned asset $@ -> $(DIST_FOLDER)$$NEW_BASE_NAME"; \
			cp $@ $(DIST_FOLDER)$$NEW_BASE_NAME; \
		fi
```

[gulp-rev]: https://github.com/sindresorhus/gulp-rev
[grunt-assets-versioning]: https://github.com/theasta/grunt-assets-versioning
