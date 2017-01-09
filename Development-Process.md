_We encourage code contributions. This page describes the process for g2core development and code contribution. We use this ourselves to keep a better record of all the changes, and keep the repo cleaner._
 
##Development Process for g2
We do development by features branches and pull requests (PRs). A feature is any code change that adds new capability of fixes a bug. Feature branches should be separate, i.e. avoid multiple new functionality in a single feature. The basic process steps are:

1. [ ] [Open an Issue for the Feature](#open-an-issue)
1. [ ] [Develop And Test The Feature](#develop-the-feature)
1. [ ] [Regression Test The Branch](#regression-test-the-branch)
1. [ ] [Pull Request](#pull-request)
1. [ ] [Close the Feature](#close-the-feature)

###Open An Issue
Here are our conventions for developing new features. Yours might be somewhat different, as sometimes you don't know if you will contribute code, so these steps may be done later, or tied directly to a PR (no new issue).

- Open an issue for the feature. Explain the reason and function. If you are going to be developing it please let us know. Remember the issue number. You will need it to label the branch. 

- Make a development branch in your repo for that issue
  - Presumably the branch is from g2/edge, but may be from elsewhere (generally not recommended)
  - Name the branch dev-XXX-featurename, where XXX is the issue number from the g2 Issues 
  - Don't worry about build numbers during this process - this will get set in the last step (close feature)

###Develop The Feature
**Development:**

- Whatever you do
- It can be useful to pull edge periodically and merge edge into your development branch. This will ensure that when you finally do a PR it will merge without issues.

**Documentation:**

- We appreciate at least some documentation for a feature that adds new capabilities. You can create a wiki page in your repo that can be ported over later. In some cases this is a modification of an existing g2 capability, so you can start there.

###Regression Test The Branch
Optionally, when you think you are ready to promote the branch run a full regression test using github/synthetos/tg_pytest. Obviously trap and correct and errors.

###Pull Request
Generate a PR to the target branch - presumably g2/edge<br>

- First ensure that the PR will merge into the target branch by merging current edge into it in your repo
- If the test build does not merge resolve merge conflicts in the new branch

Notes:

- The PR must pass full regression testing after merge or it doesn't get pushed

###Close the Issue

- We will close the issue once the PR is accepted and passes regressions
