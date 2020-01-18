# A Post Commit Git Hook for Semantic Version Updates, Docker Build, Tag, and Push

![post](../images/POSTCOMMIT.png)

In our next installment, we're going to pick up a semantic version, docker build, tag, push off the cutting room floor. This small function runs as a *post-commit*, that is the function that runs after the commit has been accepted. In the life cycle of git hooks, it runs after the pre-commit hook but before the pre-push.

```bash
#!/bin/bash

# This hook handles our deployment model. It allows us to make a commit
# that will build, tag, deploy, and push a new version of PDS to the
# container registry.

# This hook is required because Jenkins doesn't have enough
# permissions to execute these actions this.

# VARIABLES:
# SEMVER_TYPE - the type of sem ver extracted from the commit
# VERSION - the version (after being bumped) to tag and push with
# DEPLOYMENT_COMMIT - boolean check of deployment commit
# DEPLOY_STRING - "deploy-"

DEPLOY_STRING="deploy-"
REGISTRY="registry.github.com/jstone28/jstone28.github.io"

# check last commit message for [deploy-dev], [deploy-stage], [deploy-prod]
DEPLOYMENT_COMMIT=$(git log -1 --pretty=oneline --abbrev-commit)

## update VERSION file
update_version() {
    # evaluate the current version at runtime
    CURRENT_MAJOR_VERSION=$(awk -F'.' '{print FILENAME, $1}' < VERSION | awk '{$1=$1};1')
    CURRENT_MINOR_VERSION=$(awk -F'.' '{print FILENAME, $2}' < VERSION | awk '{$1=$1};1')
    CURRENT_PATCH_VERSION=$(awk -F'.' '{print FILENAME, $3}' < VERSION | awk '{$1=$1};1')

    case $SEMVER_TYPE in
        major)
            echo $((CURRENT_MAJOR_VERSION+1))."$CURRENT_MINOR_VERSION"."$CURRENT_PATCH_VERSION" > VERSION
            ;;
        minor)
            echo "$CURRENT_MAJOR_VERSION".$((CURRENT_MINOR_VERSION+1))."$CURRENT_PATCH_VERSION" > VERSION
            ;;
        patch)
            echo "$CURRENT_MAJOR_VERSION"."$CURRENT_MINOR_VERSION".$((CURRENT_PATCH_VERSION+1)) > VERSION
            ;;
    esac
}

## Git tag with VERSION representation
git_tag() {
    git tag v"$VERSION"
    git push origin --tags # must be after the deploy+1 commit
}

## commit version updates
git_commit() {
    git add . # we add anything here so that we can include CHANGELOG
    git commit -m ":rocket: version updates for deployment v${VERSION}" --no-verify
}

## docker build with version tag and latest
docker_build() {
    docker build -t "$REGISTRY":"$VERSION" -t "$REGISTRY":latest .
}

## docker push
docker_push() {
    docker push "$REGISTRY":"$VERSION"
    docker push "$REGISTRY":latest
}

#####################
# pre-push function #
#####################
if [[ "$DEPLOYMENT_COMMIT" == *"$DEPLOY_STRING"* ]]; then
    echo ""
    echo "Deployment commit detected. Performing tag, build, and push..."
    echo ""
    SEMVER_TYPE=$(echo "$DEPLOYMENT_COMMIT" | sed -e 's/.*\[\(.*\)\].*/\1/' | cut -f2- -d-)
    update_version
    VERSION=$(cat VERSION)
    git_commit
    git_tag
    docker_build
    docker_push
else
    echo ""
    echo "Deployment commit not detect."
    echo ""
fi
```

The purpose behind this git hook is as the comment describes. Jenkins, at least with the version of credentials I've been delegated, doesn't have enough runtime permissions to execute these actions. This git hook is clearly meant as pipeline logic but because of those permission, they've become repo logic. Let's walk through what's going on:

First and fore most we setup some variables and grab the commit message to store in a variable.

Next we declare a few functions to use in our main. The comments of these respective commands define their actions.

Next we declare our main function. Our first check is that the commit message defines the *deploy string*. That deploy string is the one we defined at the top of the file `deploy-` in practice, however, the conditional here is checking for anything as a delineator as well (we used `[deploy-patch/minor/major]` for example `[deploy-patch]`).

If a the deploy string isn't detectd, the whole post-commit is skipped and the message `Deployment commit not detected.` is printed after the commit.

The deploy string is unique enough that the chances of it being used in normal course are fairly low; and it discrete enough that it doesn't drastically change the commit message. An example deployment commit message might look like:

```bash
git commit -m ":sparkles: added new create/ endpoint to application [deploy-minor]"
```

Once the deploy string is detected, a deluge of commands take place:

First and foremost, we extract the Semantic Version type. This will allow up to update the `VERSION` file appropriately, with the correct Semantic Version update, based on the change made. Next we set the new version as the `VERSION` variable and commit the updated Semantic Version bump:

`git commit -m ":rocket: version updates for deployment v${VERSION}" --no-verify`

We run this command as `--no-verify` to avoid any redundant checking based on any other git hooks that may be active (note: because this commit doesn't contain the deploy string, it won't trigger this specific git hook again).

After we've committed the `VERSION` file bump, we `git tag` that version as well. After we've git tagged it, we push the tags to the remote.

In our final steps of this post-commit hook we build and tag with the `VERSION` file's contents, and finally we push to the container registry.

To Sum up We've done the following:

* Declared, in a commit message, that we're ready to deploy this version
* Bumped the Semantic Version according to that same commit message
* Updated the `VERSION` file to house the new Semantic Version
* Committed the new Semantic Version to the remote
* git tagged the latest version with the correspondingly updated `VERSION` file contents
* docker built and tagged the image with the new version
* docker push the newly built and tagged version to the remote registry

That was a lot but I hope it was helpful. I think I have one more iteration of this change from a different approach. I may add that one next. /J

[Home](../index.md)
