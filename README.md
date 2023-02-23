# Containerd Publisher

### Why is it needed?
So that consumers of the containerd api can import the api without importing the complete containerd repository. eg: cadvisor

### Does it affect the existing workflow?
No. Code will be committed only to the containerd/containerd repo. The proposed containerd/api repo will be auto populated
by the github action (every 4 hours / every commit)

### Tagging
The github action also takes care of tagging. The containerd/api repo will have corresponding tags as created in the 
containerd/containerd repo.

Context:

- [Doc](https://docs.google.com/document/d/1qD0XphQ9NtrQT934Qv38ebMRnD2VQ-CE85TfPPAbiZo/edit#heading=h.59pzk991o8bl)
- [containerd/containerd#6567](https://github.com/containerd/containerd/issues/6567)

This publisher uses a modified version of the [k8s-publishing-bot](https://github.com/kubernetes/publishing-bot), 
available at [akhilerm/publishing-bot](https://github.com/akhilerm/publishing-bot/tree/containerd-test)

### Steps for Demo :

1. We will be first  running `git-filter-repo` manually , and then pushing the changes. This is a one time task that has to be done
   by the maintainer.

    ```
    git filter-repo \
            --force \
            --commit-callback 'commit.message = commit.message + b"\nContainerd-test-commit: " + commit.original_id' \
            --path "api/" \
            --path "go.mod" \
            --path "go.sum" \
            --path "errdefs/" \
            --path "protobuf/"
    
    sed -i -e '/replace (/,/)/d' go.mod
    go mod tidy
    git add go.mod go.sum
    git commit --amend
    git remote add api-repo git@github.com:akhilerm/containerd-api-test.git
    ```

2. Push the result to the containerd/api repo

3. Make some commits in the containerd repo

4. Run the github action.
    This will extract the above mentioned paths into a new repo and push them to containerd/api

5. The same can be done for release/1.6 branches also, so that tags are also pushed.



#### NOTE
- We will have dry run mode enabled in the action to make sure it works every PR and does not break after merging the changes.
- An issue will be created and github action will comment on the issue if the publishing breaks . (same as in k8s)