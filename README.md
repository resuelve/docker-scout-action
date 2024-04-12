# Docker Scout Action

This action use `docker-scout` to analyze an image and compare it to the image generated in a previous release, this
allow to see the changes and catch possible vulnerabilities.

## Inputs

| Input                 | Required | Description                                                     |
|-----------------------|----------|-----------------------------------------------------------------|
| `docker_username`     | yes      | Docker registry username                                        |
| `docker_password`     | yes      | Docker registry password                                        |
| `docker_project_repo` | yes      | Repository on Docker Hub, for example `company/project-foo`     |
| `release_tag_regex`   | yes      | Regex to match the latest GitHub release used for comparison    |
| `image_name`          | yes      | Image name that was built and exists locally                    |
| `github_token`        | yes      | Token required to publish report into pull request as a comment |

## Usage

We can define a new job to perform this analysis

```yaml
docker-image-review:
  name: Review image againts latest release
  runs-on: ubuntu-latest
  steps:
    - name: Check out the repo
      uses: actions/checkout@v3

    - name: Setup docker image tag
      run: |
        git_hash=$(echo "${GITHUB_SHA:0:8}")
        echo "image_name=myorg/my-project:$git_hash" >> $GITHUB_ENV

    - name: Build and push Docker image
      uses: docker/build-push-action@v3
      with:
        context: .
        push: false
        tags: ${{ env.image_name }}

    - name: Run Docker Scout
      uses: resuelve/docker-scout-action@master
      with:
        docker_username: ${{ secrets.DOCKER_USERNAME }}
        docker_password: ${{ secrets.DOCKER_PASSWORD }}
        docker_project_repo: 'myorg/my-project'
        release_tag_regex: '(\d{1,3}\.){2}(\d{1,3})$'
        image_name: ${{ env.image_name }}
        github_token: ${{ secrets.GITHUB_TOKEN }}
```

Enjoy 🎉
