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

On 7/23/2018 it returned:

```
[08 Oct 16 23:01 +0000] 86f6117640ebaaffb8689e241a668218b24f4690: joeduffy
[02 Feb 17 03:33 +0000] e1d56138ec2779d0bd92c2e950791f268471ae98: kurtb
[03 Feb 17 06:03 +0000] 464bd4fe281cec0f9fa2426af15527b589d007db: lukehoban
[10 Mar 17 21:50 +0000] f32430cabd17abd6f97dfb0ffee49638cfbf427c: ericrudder
[07 Jun 17 17:52 +0000] 00ade9f28af2c5438b5436e9e622b77f60a2d49c: bforsyth927
[25 Aug 17 01:00 +0000] 9fa92c02363e25298c813b4543d5d733661ee545: ellismg
[27 Aug 17 07:38 +0000] 63df03c556d5d3051bdf9acb1da7d73c8edca073: mmdriley
[12 Sep 17 19:00 +0000] 53df23dddaa2fbe6df4479573e369b859a7de5f5: CyrusNajmabadi
[18 Sep 17 20:06 +0000] c8f315227f5bc13898e4a1e75df8e0306d23eb1e: pgavlin
[20 Sep 17 20:16 +0000] e73620d346fde36c656383894460649abeba3e64: chrsmith
[20 Oct 17 01:26 +0000] deaf4b4f8729a6247154c913bb022215919aa412: lindydonna
[20 Nov 17 07:44 +0000] cba21a78c50f89ec82efa37c7d6aafda0435259b: subhabh
[10 Jan 18 16:46 +0000] 4a76d15055ec0ca76f13a2be63ca5afe6a23701c: khyperia
[17 Jan 18 00:48 +0000] 0daf287b7ba78d54ada47b1753fcd0bb74b0a8c4: justinvp
[31 Jan 18 00:31 +0000] ff987c65e5e1822cf9742cb164013d108087cdce: swgillespie
[23 Apr 18 20:44 +0000] d583926ed1a401b4b16762c49dfefc68891eebb9: hausdorff
[13 May 18 19:48 +0000] a9f5d84996b83d677e8b8bc42b413f67ec781dfb: mehzer
[15 May 18 15:56 +0000] a423ac342e322353f1b1c2d3606012a79c65c80f: efleming18
[17 Jun 18 23:17 +0000] d380203b83074a6babe74f8d6a6083c3eba6878f: octocruise
[18 Jun 18 12:29 +0000] 683f80ded85e1ad9eaad77d449be809fc9d1df46: raiseandfall
[18 Jun 18 18:54 +0000] e0eff58969557e1d58bb263c12105a5183bacc0d: twpol
[19 Jun 18 14:21 +0000] 3995dd0a87a712087bfae2a772d2201f49def9f3: shanemcd
[21 Jun 18 17:10 +0000] b547649c4e199265ed31525cfcd9c3a445043470: robertbullen
[22 Jun 18 18:24 +0000] d6248bb1d5b4f411384a8a282409ad2b13b722a1: jen20
[22 Jun 18 21:27 +0000] c5f53395e14312a07fb676603965c0b2f219c389: mikhailshilkov
[27 Jun 18 14:50 +0000] f3b5c077bdf893ba823012b3b6c4dba0bd156220: scty-tpaoletti
[03 Jul 18 08:43 +0000] 6f2e3fcb369c0eedaa1aa82ea3ff57ba219ffc50: Frassle
[19 Jul 18 18:28 +0000] 414d0901ea8c05492eb34cae2a76ad583c21c042: Jtango18
[20 Jul 18 22:19 +0000] 376321c6f20ec492ff9ebf3982ff225f88e55076: Tirke
```