# event

set -e

echo "GWBT_COMMIT_BEFORE:  $GWBT_COMMIT_BEFORE"
echo "GWBT_COMMIT_AFTER:   $GWBT_COMMIT_AFTER"
echo "GWBT_REF:            $GWBT_REF"
echo "GWBT_TAG:            $GWBT_TAG"
echo "GWBT_BRANCH:         $GWBT_BRANCH"
echo "GWBT_REPO_CLONE_URL: $GWBT_REPO_CLONE_URL"
echo "GWBT_REPO_HTML_URL:  $GWBT_REPO_HTML_URL"
echo "GWBT_REPO_FULL_NAME: $GWBT_REPO_FULL_NAME"
echo "GWBT_REPO_NAME:      $GWBT_REPO_NAME"

#
# Cleanup before run
#
rm -rf $WORKSPACE/\.git || true
rm -rf $WORKSPACE/* || true
cd $WORKSPACE

#
# Prevent manual Job starts
#
if [[ -z "$GWBT_COMMIT_AFTER" ]]
then
    echo "I DON'T WANT JOBS STARTED MANUALLY! ONLY VIA GITHUB WEBHOOK!"
    exit 1
fi


#
# Only Build Branches
#
if [ -z "$GWBT_BRANCH" ]
then
	echo "THIS PUSH IS NOT INSIDE A BRANCH. I DON'T LIKE IT!"
    exit 1
fi

#
# Clone specific branch
#
git clone --single-branch \
          --branch $GWBT_BRANCH \
          https://github.com/${GWBT_REPO_FULL_NAME}.git \
          source

#
# Switch to specific revision
#
cd source
git reset --hard $GWBT_COMMIT_AFTER

#
# Trigger build script inside cloned repository
#
bash jenkins.sh
