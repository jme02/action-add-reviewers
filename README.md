This is a PoC demonstrating how we could assign reviewers to a PR dependant on the changes within.

TESTING

There already exist various plugins on the github marketplace we could use, however they all do a very similar thing:
 - take configuration via input
 - check the diff for changed files
 - use the config and the changes to detirmine who should be assigned as a reviewer
 - post to the github api and update the reviewers

This is simple to do ourselves and saves us getting these plugins approved for use in Flutter.

The .github/workflows/pr-assign.yml file contains this logic and is broken up into the steps above:
 - The files changed in the PR are detirmined using git diff https://github.com/jme02/action-add-reviewers/blob/main/.github/workflows/pr-assign.yml#L27. A pre-requisite of this is that the fetch-depth is set to '2' otherwise no changes will be found. Additionally the output is saved in the `GITHUB_OUTPUT` so it can be accessed in other steps, but first it needs the newlines removing and converting to a comma seperated format.
- We can insert the output of the git diff directly into some JavaScript as a string like so https://github.com/jme02/action-add-reviewers/blob/main/.github/workflows/pr-assign.yml#L35. 
- We can easily read in a configuration file https://github.com/jme02/action-add-reviewers/blob/main/.github/workflows/pr-assign.yml#L37
- Now we have both the changed files and the configuration file we can do whatever we want. In this PoC we take a group and a corresponding regex from the config and check it against all the changes, if there is a match we can return this group as an output of the script. For multiple matches we can also do anything, in this PoC we just return a set list.
- The output of the script can be used as part of the cURL https://github.com/jme02/action-add-reviewers/blob/main/.github/workflows/pr-assign.yml#L60 to set the reviewers on the PR. The other variables like PULL_NUMBER and GITHUB_TOKEN are standard variables available to a github actions.

The end functionality of this PoC is a github action that:
 - can assign reviewers based on the files changed in the PR
 - can have different behaviour defined in cases of overlapping changes I.E multiple unrelated projects with different groups of reviewers changed
 - can have behaviour defined for no matches
 - readable and flexible configuration format stored in the `contributors.json`

Future
The use of the github api can be extended and used to perform other actions on the PR based on files changed.
