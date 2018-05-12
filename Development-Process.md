_We encourage code development and contribution. This page describes the process for g2core development and code contribution. We use this ourselves to keep a better record of all the changes, and keep the repo cleaner._

See also: [Getting Started with g2core Development](https://github.com/synthetos/g2/wiki/Getting-Started-with-g2core-Development)

## Development Process for g2
We do development by features branches and pull requests (PRs). A feature is any code change that adds new capability of fixes a bug. Feature branches should be separate, i.e. avoid multiple new functionality in a single feature. The basic process steps are:

1. [ ] [Open an Issue for the Feature](#open-an-issue)
1. [ ] [Develop And Test The Feature](#develop-the-feature)
1. [ ] [Regression Test The Branch](#regression-test-the-branch)
1. [ ] [Pull Request](#pull-request)
1. [ ] [Close the Feature](#close-the-feature)
1. [ ] [Tag a Release](#tag-a-release)
1. [ ] [Publish the Release and Release Notes](#publish-release-and-release-notes)

### Open An Issue
Here are our conventions for developing new features. Yours might be somewhat different, as sometimes you don't know if you will contribute code, so these steps may be done later, or tied directly to a PR (no new issue).

- Open an issue for the feature. Explain the reason and function. If you are going to be developing it please let us know. Remember the issue number. You will need it to label the branch.

- Make a development branch in your repo for that issue
  - Presumably the branch is from g2/edge, but may be from elsewhere (generally not recommended)
  - Name the branch dev-XXX-featurename, where XXX is the issue number from the g2 Issues
  - Don't worry about build numbers during this process - this will get set in the last step (close feature)

### Develop The Feature
**Development:**

- Whatever you do
- It can be useful to pull edge periodically and merge edge into your development branch. This will ensure that when you finally do a PR it will merge without issues.

**Documentation:**

- We appreciate at least some documentation for a feature that adds new capabilities. You can create a wiki page in your repo that can be ported over later. In some cases this is a modification of an existing g2 capability, so you can start there.

### Regression Test The Branch
When you think you are ready to promote the branch you should run a full regression test using github/synthetos/tg_pytest. Obviously trap and correct and errors. All instructions are in the readme.

### Pull Request
Generate a PR to the target branch - presumably g2/edge<br>

- First ensure that the PR will merge into the target branch by merging current edge into it in your repo
- If the test build does not merge resolve merge conflicts in the new branch
- Do the pull request: - https://github.com/synthetos/g2/pulls
  - Go to [Compare](https://github.com/synthetos/g2/compare) and hit `compare across forks` to pull from your repo

Notes:

- The PR must pass full regression testing after merge or it doesn't get pushed

### Close the Issue

- We will close the issue once the PR is accepted/ We may advance the build number as well, depending on the nature of the PR.

### Tag a Release

- After a group of PRs are closed, and the code in `edge` is determined to be at least usable for testing, then we can tag it, which will trigger TravisCI to make a build and push the binaries to to [releases](https://github.com/synthetos/g2/releases) page.

- The tagging process:

  - In a commit on it's own, change the define [`G2CORE_FIRMWARE_BUILD`](https://github.com/synthetos/g2/blob/edge/g2core/g2core_info.h#L24) in g2core_info.h to use the format described next, with a one-sentence brief description of what's in the change. You can refer to fixed issues by the github convention of `#nnn`.

  - For now, tags must take the form of `XXX.nn` where XXX is currently `101` and `nn` is updates by 1 each tag. It is important that there not be any additional spaces or letters in the tag itself. The git tag itslef is an "annotated" and unsigned tag with the comment/description on the tag should match the comment in the code.

  - For example, the code would say:
      ```c++
      #define G2CORE_FIRMWARE_BUILD 101.01 // fixes #123, #124, and #125
      ```
  
      and the git commit line would read:
  
      ```bash
      git tag -a 101.01 -m '// fixes #123, #124, and #125'
      ```

### Publish Release and Release Notes

- When satisfied with the tag, push it to Origin (or whatever your target is). This will kick off the Travis build that creates the release on the releases page.

- Once that's done, go to the release page and create / edit the release notes. This should just be a cut and paste of the latest section of the readme. but may have additional information as well. 
