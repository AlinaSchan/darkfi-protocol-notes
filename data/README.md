# data

the files behind [development-history.md](../notes/development-history.md), as
[repository-history-tools](https://github.com/alinaschanz/repository-history-tools) wrote them on
2026-09-15 against the github copy of darkfi at commit `a5514cb2cecdc791ed8de68440455433beeebe7a`:

```
python history.py /path/to/darkfi --output ./data
```

- `summary.json`: head, first commit, counts, the options used
- `commits_by_month.csv`: commits, merges, authors, files changed, insertions, deletions per month
- `most_changed_directories.csv`: directories two levels deep, by commits
- `first_last_change.csv`: top-level directories, first and last change, whether still present
- `repository_growth.csv`: files and bytes in the tree at the end of each month on the mainline
- `most_changed_paths.csv`: every path that ever changed, by commits

`contributors.csv` is left out on purpose: it is a list of people's names, and the notes only use
the counts from it.

rename detection is off and merges carry no file changes; the tool's readme explains what that does
to the numbers. rerunning the command against the same commit should reproduce the files except for
`generated_at` and `git_version` in `summary.json`.
