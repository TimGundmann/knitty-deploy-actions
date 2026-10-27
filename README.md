# Shared KnittyNetwork deployment actions

`scp-with-retry/action.yml` is the central file-transfer action for the five
application services and three frontends in this workspace. It uses the existing
`appleboy/scp-action@v1.0.0`, with a maximum of three attempts and a 30-second pause
after each failed attempt. A successful transfer skips the remaining attempts.
If all attempts fail, the action fails and deployment stops.

Only transfers are retried. Builds, tests, remote deployment commands and rollback
commands are not repeated. A transfer can overwrite files already extracted by a
partially completed attempt; frontend transfers retain their existing in-place
deployment behavior.

## Rollout

The action is published from `TimGundmann/knitty-deploy-actions` on `main`.
The repository contains deployment logic only; never commit credentials.
Public visibility is needed because the public `config-service` repository
also uses this action.

Merge the consumer workflow pull requests in `bff`, `newsletter-service`,
`pattern-service`, `user-service`, `config-service`, `frontend`, `knottynetwork`
and `knottynetwork-frontend` to enable retries. Their existing push-to-default-branch
triggers then run their deployment workflows. Publishing this action alone does
not change any running service.

Each consumer passes its existing SSH secrets, source and destination:

```yaml
- name: Transfer deployment files with retry
  uses: TimGundmann/knitty-deploy-actions/scp-with-retry@main
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SERVER_SSH_KEY }}
    source: dist
    target: /var/www/example
    strip_components: '1'
```

`strip_components` defaults to `0` for service deployments. All callers reference
`main`, so future changes to retry behavior are maintained in this repository and
take effect on their next workflow run.

The API/client libraries publish packages rather than transferring files over SSH
and do not use this action. `feed-service` has no deployment workflow to update.
