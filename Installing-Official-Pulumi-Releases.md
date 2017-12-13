We have a script, [`install-pulumi.sh`](https://github.com/pulumi/home/blob/master/scripts/install-pulumi.sh) that is checked in to the scripts folder in pulumi/home, which we use during CI to install the SDK on our worker machines. We also give this script to customers so they can install Pulumi as part of their CI runs.

Because we don't want to share the bits publicly yet, we require you have a Pulumi Access Token to access them. To get yours, look at your [account](https://beta.pulumi.com/account) page on beta.pulumi.com.

Once you have it, from your shell in `pulumi/home` run:

```
PULUMI_ACCESS_TOKEN="<your-acccess-token-goes-here>" ./scripts/install-pulumi.sh v0.9.2
```

You can, of course, select a different version of the SDK, if you'd like.

This will install Pulumi into `/usr/local/pulumi` and then link all the Pulumi modules to point at the versions from that SDK. The Pulumi that is built for you normally (in `/opt/pulumi`) is not touched.  To use this Pulumi, add `/usr/local/pulumi/bin` to the start of your `$PATH`.  To switch back to your source built version, just remove it from your path and then run `make only_build` in each of the pulumi repositories (this will rebuild and relink everything).