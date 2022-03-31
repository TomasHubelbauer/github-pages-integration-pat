# GitHub Pages Integration PAT

GitHub has a feature called GitHub Pages. This feature allows having GitHub host
a website based on the contents of a repository on a github.io domain or on a
custom domain.

GitHub Pages get deployed with every commit to the repository, so they stay up
to date with the repository contents automatically.

GitHub also has a feature called GitHub Actions. Actions is a CI/CD solution
built into GitHub itself. The only thing one needs to do is to commit a YAML
file called a workflow file into a well-known path within the repository and
GitHub Actions will kick in.

Another thing GitHub has is an API. You use a personal access token (a PAT) to
call the API. Various API areas require different scopes on the PAT you are
using to talk to the GitHub API. Another thing you can do with the PAT is to
authenticate with GitHub for the purposes of using the Git API GitHub exposes.

Putting this all together, it is possible to have a GitHub repository which
makes changes to the checked out repository, sets up a Git identity, commits
the changes, authenticates with GitHub using a PAT and pushes the changes back
to the repository.

One way GitHub Actions know not to run again for the push made from the Actions
workflow run is because in GitHub Actions, there is an automatic PAT provided
which has a limited set of scopes and can be used to identify actions made by
automation as opposed to real users. It also has a separate set of rate limits
as far as I remember.

By the same token, GitHub Actions knows to run when you do use your real token
to push changes to your repository. And GitHub Pages is just a special, hidden
GitHub Actions workflow that also runs on push with a custom token. Neither will
run on changes made by the automatic (also called "integration" token).

This is all good and well, but what if you actually want to publish a change to
GitHub Pages resulting from a push to the repository that came from the GitHub
Actions pipeline?

One way is to use a custom PAT token. It is not wise to push with a custom PAT,
because GitHub Actions will lose the ability to tell it was the integration that
pushed to the repository and it will trigger itself in an infinite loop. That's
what I think at least. You could use the "no ci" keyword in your commit message
from the workflow with a custom PAT to prevent queueing of another run, but
that's hack teritory if you can just use the integration token. But then, you
won't have GitHub Pages deploying on the push…

Another way is to still use a custom PAT, but not to push a change and trigger a
GitHub Pages deployment through it, but instead to call the GitHub API with the
custom PAT and making sure it has the right scopes to actually be able to call
the GitHub Pages API for deploying a new version of the site based on the latest
contents of the repository.

Everything I have described so far is documented in my repository at
https://github.com/tomashubelbauer/github-actions

Recently, I learnt that it is possible to grant the automation integration PAT
extra privileges, and to even do it step-wise. More on that here in the GitHub
documentation:
https://docs.github.com/en/actions/security-guides/automatic-token-authentication#modifying-the-permissions-for-the-github_token

GitHub PAT are currently going through some sort of a transition from an old
style of a token to a new style of a token and it seems a change in naming of
the scopes is coming along with it. The GitHub API documentation still uses the
old names of the scopes, but the new names are similar to what the GitHub App
uses for scope names. So there is not exactly a 1:1 match between scope names
as listed in the various API documentation pages and what scopes you can set on
your integration PAT in the workflow file. But you can tweak the scopes:
https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token

With all this in mind, it should be possible to use the integration token scope
extension in the workflow file to grant the token the right to the scope for
GitHub Pages. And so, to allow the integration token to call the GitHub Pages
API and queue a deployment without having to make a custom PAT for every repo
where this is desirable.

This used to be more dangerous than it is now, because nowadays PATs can get
scoped to a particular repository, so a leak of a PAT would give a malicious
finder only the ability to influence the associated repository, not all repos
of the token issuer user. Still, it is not ideal to go minting an extra token
for every repository. This mechanism I am wondering about here should help
tremendously.

## To-Do

### See if it is possible to call the GitHub Pages API with integration PAT

In the workflow file, add the PAT the `pages` read/write scope so that it can 
use the API and see if it works. If so, updated the `github-pages` repository
of mine to codify this best practice.

### See if using a custom PAT to push a change to the workflow repository loops

I think this must be the case, but it would be good to have a proof for the
claim.

### See if the integration token is automatically scoped to the single repo

I expect this will be the case, but it would still be good to have a proof of
this.
