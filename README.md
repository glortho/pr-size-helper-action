# PR Size Helper action

This action adds [size labels](https://github.com/kubernetes/kubernetes/labels?q=size) to pull requests. If a pull requests is above a configured complexity threshold (calculated by default by summing lines of additions and deletions), the action will prompt the PR author for more context to help explain the size of the PR. If the author chooses to give additional context, the reason will be tracked along with all others in a digest issue.

## Demo

**1. A contributor creates a new PR:**

![a large pr is created](https://user-images.githubusercontent.com/1746081/112671818-e7432600-8e1f-11eb-8ca4-d6849eb77b14.png)

**2. pr-size-helper-action labels it with a (configurable) `size/` label:**

![pr is labeled with size label](https://user-images.githubusercontent.com/1746081/112671828-ee6a3400-8e1f-11eb-9225-e3021fc31896.png)

**3. If the PR crosses a (configurable) complexity size threshold the PR creator is prompted with a comment on the PR like this:**

<img width="841" alt="Screenshot 2023-05-16 at 2 07 02 PM" src="https://github.com/glortho/pr-size-helper-action/assets/526284/567ae55d-0eac-4355-94db-9705d1ce7a33">

## Usage

Create the workflow file:

`.github/workflows/apply-pr-size-label.yml`

```
name: Apply PR size label

on: pull_request

jobs:
  apply_pr_size_label:
    runs-on: ubuntu-latest
    steps:
      - uses: levindixon/pr-size-helper-action@v1.5.0
        env:
          GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

## Complexity scoring

By default, the complexity score is calculated as follows:

```
Lines added + lines removed - whitespace lines - comment lines = complexity score.
```

Note that `IGNORED` files are not factored into the calculation.

There are additional scoring strategies that you can opt into if you choose:

- `tests-are-less-complex`: Subtract 0.5 points for each line in a test file.
- `single-words-are-less-complex`: Subtract 0.5 points for each line that has nothing on it but a single word.

See the `SCORING_STRATEGIES` variable below for details on how to opt into these.

## Configuration

The following environment variables are supported:

- `IGNORED`: A list of [glob expressions](http://man7.org/linux/man-pages/man7/glob.7.html)
  separated by newlines. Files matching these expressions will not count when
  calculating the complexity of the pull request. Lines starting with `#` are
  ignored and files matching lines starting with `!` are always included.

- `PROMPT_THRESHOLD`: Pull requests created with a complexity score greater or equal to this value will trigger a friendly message prompting the pull request author to provide a reason for the size of the pull request. Defaults to 500.

- `S` | `M` | `L` | `XL` | `XXL`: Setting one, some, or all of these will change the pull request size labelling. Pull requests with a complexity score between 0 and `S` will be labeled as `size/XS`, PRs with a size between `S` and `M` will be labeled as `S` and so on. Defaults:
  - `S`: 10
  - `M`: 30
  - `L`: 100
  - `XL`: 500
  - `XXL`: 1000

- `AUTHOR_LOGINS`: This is a space-delimited list of GitHub usernames matching PR authors for whom this action will run. Any PRs authored by users not included in this list (or part of an enabled team -- see below) will be skipped.

- `TEAMS`: (Note: Not yet functional.) This a space-delimited string of [team](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams) slugs, that restricts
this workflow from running any time that the PR author is not a member of one of
the specified teams. The owning organization for each team is assumed to be the
the owner of the repository that acts as the base of the newly opened pull request.

- `SCORING_STRATEGIES`: An optional space-delimited list of strategies to use when calculating the complexity of a PR:
  - `tests-are-less-complex`: This strategy subtracts 0.5 points for each line change in test files.
  - `single-words-are-less-complex`: This strategy subtracts 0.5 points for each line change where the content of the line being changed is a single word. 

You can configure the environment variables in the `apply-pr-size-label.yml` workflow file like this:

```yaml
env:
  GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
  IGNORED: ".*\n!.gitignore\nyarn.lock\ngenerated/**"
  PROMPT_THRESHOLD: 500
  S: 10
  M: 30
  L: 100
  XL: 500
  XXL: 1000
  SCORING_STRATEGIES: "tests-are-less-complex"
```

## Acknowledgments

- This was originally forked from [`levindixon/pr-size-helper-action`](https://github.com/levindixon/pr-size-helper-action). Thank you @levindixon! ✨

- 📝 Repo templated using [`actions/javascript-action`](https://github.com/actions/javascript-action) 📝

- ✨ Guided by the [Creating a JavaScript action](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action) guide ✨

- 🙇‍♂️ Ignore and labeling functionality forked from [`pascalgn/size-label-action`](https://github.com/pascalgn/size-label-action) 🙇‍♂️

- 💬 Prompt inspiration from [`CodelyTV/pr-size-labeler`](https://github.com/CodelyTV/pr-size-labeler) 💬

- 🏷 Size labels borrowed from [`kubernetes/kubernetes`](https://github.com/kubernetes/kubernetes/labels?q=size) 🏷

## License

[MIT](LICENSE)
