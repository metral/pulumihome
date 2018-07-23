This query gets the earliest contributions for every user who has contributed to any Pulumi project.

```go
package main

import (
	"context"
	"fmt"
	"sort"
	"testing"
	"time"

	"github.com/google/go-github/github"
	"golang.org/x/oauth2"
)

const accessToken = "YOURGITHUBACCESSTOKEN"

func TestGetEarliestCommitsForAllPulumiContributors(t *testing.T) {
	ctx := context.Background()
	ts := oauth2.StaticTokenSource(
		&oauth2.Token{AccessToken: accessToken},
	)
	tc := oauth2.NewClient(ctx, ts)
	client := github.NewClient(tc)
	repos, _, err := client.Repositories.ListByOrg(ctx, "pulumi", nil)
	if err != nil {
		t.Fail()
	}

	earliestCommits := make(map[string]*github.RepositoryCommit)

	for _, repo := range repos {
		if repo.Private != nil && *repo.Private {
			continue
		}
		if repo.GetName() == "terraform-provider-aws" || repo.GetName() == "terraform" {
			continue
		}
		stats, _, err := client.Repositories.ListContributorsStats(ctx, "pulumi", repo.GetName())
		if err != nil {
			t.Fail()
		}
		for _, stat := range stats {
			author := stat.GetAuthor().GetLogin()
			_, resp, err := client.Repositories.ListCommits(ctx, "pulumi", repo.GetName(), &github.CommitsListOptions{
				Author: author,
				ListOptions: github.ListOptions{
					PerPage: 5,
				},
			})
			if err != nil {
				t.Fail()
			}
			lastPage := resp.LastPage
			commits, resp, err := client.Repositories.ListCommits(ctx, "pulumi", repo.GetName(), &github.CommitsListOptions{
				Author: author,
				ListOptions: github.ListOptions{
					PerPage: 5,
					Page:    lastPage,
				},
			})
			if len(commits) == 0 {
				fmt.Printf("No commits for %s and %s (last page %d)!", repo.GetName(), author, lastPage)
			}
			firstcommit := commits[len(commits)-1]
			if previous, ok := earliestCommits[author]; ok {
				if firstcommit.GetCommit().GetCommitter().GetDate().Before(previous.GetCommit().GetCommitter().GetDate()) {
					earliestCommits[author] = firstcommit
				}
			} else {
				earliestCommits[author] = firstcommit
			}
		}
	}

	var arr []*github.RepositoryCommit

	for _, commit := range earliestCommits {
		arr = append(arr, commit)
	}

	sort.Slice(arr, func(i, j int) bool {
		return arr[i].GetCommit().GetCommitter().GetDate().Before(arr[j].GetCommit().GetCommitter().GetDate())
	})

	for _, commit := range arr {
		fmt.Printf("[%s] %s: %s\n", commit.GetCommit().GetCommitter().GetDate().Format(time.RFC822Z), commit.GetSHA(), commit.GetAuthor().GetLogin())
	}
}

```
