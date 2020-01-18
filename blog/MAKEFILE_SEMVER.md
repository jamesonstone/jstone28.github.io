# A Semantic Versioning Function for the Makefile

![make](../images/MAKE_SEMVER.png)

I've been spending a lot of time in Jenkins lately; most certainly against my will. Jenkins offers all of the downfalls of a decentralized open source project:

* Disjointed road map
* Feature incomplete
* Difficult to decipher UI
* Multiple implementations of the same feature
* Redundant methods within a single feature
* Much much more!

It's because of this that I've had to move a lot of the logic I would normally keep in the pipeline, into the project itself. Such as Semantic Versioning Function:

```Makefile
.PHONY: increment-version
increment-version:
	@echo -n "Add CHANGELOG entry? Y to exit for entry creation [Y/n] " && read ans && [ $${ans:-N} = n ]
	@# evaluate the current version at runtime
	$(eval CURRENT_MAJOR_VERSION := $(shell awk -F'.' '{print FILENAME, $$1}' < VERSION))
	$(eval CURRENT_MINOR_VERSION := $(shell awk -F'.' '{print FILENAME, $$2}' < VERSION))
	$(eval CURRENT_PATCH_VERSION := $(shell awk -F'.' '{print FILENAME, $$3}' < VERSION))
# major
ifeq ($(SEMVER),$(filter $(SEMVER), MAJOR major))
	@echo $$[$(CURRENT_MAJOR_VERSION)+1].$(ZERO).$(ZERO) > VERSION
endif
# minor
ifeq ($(SEMVER),$(filter $(SEMVER), MINOR minor))
	@echo $(CURRENT_MAJOR_VERSION).$$[$(CURRENT_MINOR_VERSION)+1].$(ZERO) > VERSION
endif
# patch
ifeq ($(SEMVER),$(filter $(SEMVER), PATCH patch))
	@echo $(CURRENT_MAJOR_VERSION).$(CURRENT_MINOR_VERSION).$$[$(CURRENT_PATCH_VERSION)+1] > VERSION
endif
	@echo "version updated."
```

The first prompt asks the user if they want to enter a `CHANGELOG` entry. If they user answers yes, the make command is exited with an error. This allows the user to make the change to the `CHANGELOG` before the versioning takes place.

Alternatively, if the user answers that they do not need to enter a `CHANGELOG` entry, the rest of the command will execute.

This function works in conjunction with a `VERSION` file which contains a plain Semantic Version such as `0.0.3`

```bash
make increment-version SEMVER=patch/minor/major
```

The type of change is set to the `SEMVER` variable and that prompts the update mechanism to make the corresponding change. For example:

If the current version is `1.0.3` and we run the following command:

```bash
make increment-version SEMVER=patch
```

The resulting version in the `VERSION` file will be `1.0.4`. If we again run the increment-version command:

```bash
make increment-version SEMVER=minor
```

The resulting version then will be `1.1.0`. If we run the increment-version command a final time:

```bash
make increment-version SEMVER=major
```

The `VERSION` file will read `2.0.0`

I hope this snippet is useful to someone. I didn't spend too much time putting it together but it's, at least, the 10th time I've written something like this.

[Home](../index.md)
