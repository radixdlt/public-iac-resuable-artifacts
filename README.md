# public-iac-resuable-artifacts

WARNING: These actions are not useful for developers outside the radixdlt, radixworks, metaverse or metapass organisation as the workflows and steps are highly specialized for usage in the company. Since some repositories are public, they are not able to access private or internal repositories.

Reusable workflows:
.github/workflows/

## Docker build and push

There is a variety of settings and steps that are done to perform a docker push and this reusable step aims to both simplify the process for developers aswell as improve the process. Developers profit from reduced effort to write github actions, additional features often not used yet like image scanning and linting and also improve their build times with mandatory caching.
Devops and Security profit from a single point of maintenance in case of required updates for github actions.

### Parameters

#### image_registry, image_organization, image_name, tag, labels, platforms

Hopefully self-explaining parameters to specify the docker image tag. 
```
tags: |
  ${{ inputs.image_registry }}/${{ inputs.image_organization }}/${{ inputs.image_name }}:${{ inputs.tag }}
```

`image_name` is also used to specify the name of the cache-repo.

For dockerhub specify `registry.hub.docker.com` as `image_registry`.

#### cache_tag_suffix 
Cache is always pushed to the cache artifact registry. This can only be configures by changing the cache tag suffix.
Use this to seperate workstreams that are vastly different from each other and tend to disrupt each others cache. 
That results in longer build times for both.

We recommend a suffix for each stage (dev,pr,release).

```
cache-from: |
type=registry,ref=europe-west2-docker.pkg.dev/dev-container-repo/eu-cache-repo/${{ inputs.image_name }}:build-cache-${{ inputs.cache_tag_suffix }} 
cache-to: |
type=registry,ref=europe-west2-docker.pkg.dev/dev-container-repo/eu-cache-repo/${{ inputs.image_name }}:build-cache-${{ inputs.cache_tag_suffix }},mode=max 
```

#### restore_artifact, artifact_name, artifact_location
These parameters come together to allow downloading of pre build artifacts for copying into the docker image. 
Use this if your docker build requires local files.

```
# Restoring Artifacts build in a previous step
    - if: inputs.restore_artifact == 'true'
    name: Download artifact
    uses: actions/download-artifact@v3
    with:
        name: ${{ inputs.artifact_name }}
        path: ${{ inputs.artifact_location }}
    - if: inputs.restore_artifact == 'true'
    name: Display structure of downloaded files
    run: ls -R ${{ inputs.artifact_location }}
```

#### authentication

Authentication to Google Cloud is mandatory and requires two secrets to be available to Github. GCP_WORKLOAD_IDP and GCP_SERVICE_ACCOUNT. The values of these should be the same in each repository. Authorization for each repository is done in the Google Cloud console. Contact @team-devops in Slack to add your new Repo.

In case of a public image push, you will need docker credentials. These require the use of environment, enable_dockerhub and specify the role_to_assume. The role to assume will always be the same and will move to secrets in the future. The environment should be specified as release and be a protected environment that only allows access to protected branches. The AWS Role enforces this. This prevents pull_requests from malicious parties to push images to our public docker registry.

```    
    environment: "release"
    role_to_assume: "arn:aws:iam::1234567890:role/rolename"
    enable_dockerhub: "true"
  secrets:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDP }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```