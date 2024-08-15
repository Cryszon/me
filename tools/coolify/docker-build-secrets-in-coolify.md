# Docker build secrets in Coolify

Coolify doesn't currently _(2024-08-15)_ support build secrets.

> After some more research, testing and looking at [Coolify source
> code](https://github.com/coollabsio/coolify/blob/69fc4c7f525100c9291b75f4b9f31f92b285bf14/app/Jobs/ApplicationDeploymentJob.php#L427),
> I've come to the conclusion that this is not currently possible. Coolify seems
> to handle env variables defined in the UI by writing them into an env file
> that is then used in the `environment` configuration for each service, which
> means that the top level `secrets` element won't be able to access these
> variables.
>
> This means that support for build secrets needs to be added as a new feature.
> Secrets could still be managed in Coolify through the Environment Variables
> UI, but they need to be passed to Docker separately.
>
> Feel free to correct me if I'm wrong.
>
> -- <cite>[@Cryszon on Coolify
> Discord](https://discord.com/channels/459365938081431553/1273315947893096470)</cite>

Here's a workaround using build args insead of secrets.

> [!CAUTION]
>
> [Build variables are not a secure replacement for
> secrets](https://docs.docker.com/build/building/variables/)! Using them makes
> the secret visible in the final built image. `sh -c '...'` is used to hide the
> secret from the build logs and the temp file is removed (`rm /tmp/...`), but
> the secret value will still be readable by inspecting the built image.
>
> See also [this
> documentation](https://docs.docker.com/reference/dockerfile/#arg) and [this
> discussion](https://stackoverflow.com/q/44615837/1865857)

Let's say you use a secret in your Dockerfile:

```Dockerfile
# Dockerfile
RUN --mount=type=secret,id=test_secret \
  echo "Command that uses secret from /run/secrets/test_secret"
```

This is how you would use build arg instead:

```Dockerfile
# Dockerfile
ARG test_secret
RUN sh -c 'echo "${test_secret}" > /tmp/test_secret' && \
    echo "Command that uses secret from /tmp/test_secret"
RUN rm /tmp/test_secret
```

## How to simulate Coolify deployment build args locally

When you check **Build Variable?** in Coolify UI, Coolify passes it as a build
argument to the build command. You can simulate this by using `--build-arg` when
building locally. The recommended way to test this is to use a separate file for
storing the secret.

```sh
docker compose build --build-arg test_secret=$(cat secret_file)
```

