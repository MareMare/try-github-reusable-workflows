# try-github-reusable-workflows
 🧪 Reusing workflows のおためし

## カスタムな GitHub Actions を使用する方法

<details>
<summary><code>jobs.<job_id>.steps[*].uses</code> で指定する例：</summary>

「[jobs\.<job\_id>\.steps\[\*\]\.uses]」には、次の記載があります。
> ジョブのステップの一部として実行するアクションを選択します。アクションは、再利用可能なコードの単位です。**ワークフローと同じリポジトリ、パブリック リポジトリ**、または**公開された Docker コンテナ イメージ**で定義されたアクションを使用できます。

1. [Example: Using a public action] `{owner}/{repo}@{ref}`
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: My first step
          # Uses the default branch of a public repository
          uses: actions/heroku@main
        - name: My second step
          # Uses a specific version tag of a public repository
          uses: actions/aws@v2.0.1
    ```
2. [Example: Using a public action in a subdirectory] `{owner}/{repo}/{path}@{ref}`
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: My first step
          uses: actions/aws/ec2@main
    ```
3. [Example: Using an action in the same repository as the workflow] `./path/to/dir`
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: Check out repository
          uses: actions/checkout@v3
        - name: Use local my-action
          uses: ./.github/actions/my-action
    ```
4. [Example: Using a Docker Hub action] `docker://{image}:{tag}`
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: My first step
          uses: docker://alpine:3.8
    ```
5. [Example: Using the GitHub Packages Container registry] `docker://{host}/{image}:{tag}`
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: My first step
          uses: docker://ghcr.io/OWNER/IMAGE_NAME
    ```
6. [Example: Using a Docker public registry action] `docker://{host}/{image}:{tag}`
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: My first step
          uses: docker://gcr.io/cloud-builders/gradle
    ```

</details>

<details>
<summary>ワークフローとは異なる private リポジトリ内のアクションを使用する例：</summary>

どうやら、ワークフローのリポジトリに private リポジトリを取り込んで action を使用する模様。
* [Example: Using an action inside a different private repository than the workflow]
    > ワークフローでは、プライベートリポジトリをチェックアウトし、アクションをローカルで参照する必要があります。個人用アクセストークンを生成し、そのトークンを暗号化されたシークレットとして追加します。詳しい情報については「[個人用アクセストークンの作成]」および「[暗号化されたシークレット]」を参照してください。
    ```yml
    jobs:
      my_first_job:
        steps:
        - name: Check out repository
          uses: actions/checkout@v3
          with:
            repository: octocat/my-private-repo
            ref: v1.0
            token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
            path: ./.github/actions/my-private-repo
        - name: Run my action
          uses: ./.github/actions/my-private-repo/my-action
    ```

</details>

<details>
<summary>ワークフローとは異なる public リポジトリ内の reusable workflow を使用する例：</summary>

[Reusing workflows \- GitHub Docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows) によると再利用可能なワークフローを纏めたリポジトリから直接使用することができるらしい。

* 再利用可能なワークフローの例
  ```yml
  name: Reusable workflow example

  on:
    workflow_call:
      inputs:
        config-path:
          required: true
          type: string
      secrets:
        token:
          required: true

  jobs:
    triage:
      runs-on: ubuntu-latest
      steps:
      - uses: actions/labeler@v4
        with:
          repo-token: ${{ secrets.token }}
          configuration-path: ${{ inputs.config-path }}
  ```

* 再利用可能なワークフローの呼び出し例
  ```yml
  name: Call a reusable workflow

  on:
    pull_request:
      branches:
        - main

  jobs:
    call-workflow:
      uses: octo-org/example-repo/.github/workflows/workflow-A.yml@v1

    call-workflow-passing-data:
      permissions:
        contents: read
        pull-requests: write
      uses: octo-org/example-repo/.github/workflows/workflow-B.yml@main
      with:
        config-path: .github/labeler.yml
      secrets:
        token: ${{ secrets.GITHUB_TOKEN }}
  ```

</details>

<!-- リンク -->
[jobs\.<job\_id>\.steps\[\*\]\.uses]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsuses

[Example: Using a public action]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-public-action

[Example: Using a public action in a subdirectory]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-public-action-in-a-subdirectory

[Example: Using an action in the same repository as the workflow]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-an-action-in-the-same-repository-as-the-workflow

[Example: Using a Docker Hub action]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-docker-hub-action

[Example: Using the GitHub Packages Container registry]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-the-github-packages-container-registry

[Example: Using a Docker public registry action]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-docker-public-registry-action

[Example: Using an action inside a different private repository than the workflow]: https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#example-using-an-action-inside-a-different-private-repository-than-the-workflow

[個人用アクセストークンの作成]: https://docs.github.com/ja/github/authenticating-to-github/creating-a-personal-access-token
[暗号化されたシークレット]: https://docs.github.com/ja/actions/reference/encrypted-secrets

## Deprecating save-state and set-output commands な件

2022/10/11 から非推奨になったみたい。
* [GitHub Actions: Deprecating save\-state and set\-output commands \| GitHub Changelog](https://github.blog/changelog/2022-10-11-github-actions-deprecating-save-state-and-set-output-commands/)

❌ いままで
```yml
- name: Save state
  run: echo "::save-state name={name}::{value}"

- name: Set output
  run: echo "::set-output name={name}::{value}"
```

✅ これから
```yml
- name: Save state
  run: echo "{name}={value}" >> $GITHUB_STATE

- name: Set output
  run: echo "{name}={value}" >> $GITHUB_OUTPUT
```

## 条件に応じて [jobs.<job_id>.runs-on] を指定する方法

❌ `runs-on: ${{ inputs.runs-on }}` などと指定できない

<details>
<summary>syntax errorになるワークフローの例:</summary>
<div>

```yml
name: 09.choice-runner-os
on:
  workflow_dispatch:
    inputs:
      runs-on:
        description: 'OS'
        required: true
        defaults: "ubuntu-latest"
        type: choice
        options:
          - "ubuntu-latest"
          - "windows-latest"
jobs:
  sample:
  runs-on: ${{ inputs.runs-on }} # ❌ `Invalid workflow file. You have an error in your yaml syntax on line 15`
  steps:
    - name: おためし
      run: echo "runs on ${{ inputs.runs-on }}"
```

</details>

✅ `runs-on: ${{ needs.setup.outputs.runner }}` などで指定できる

[Specify runner to be used depending on condition in a GitHub Actions workflow \- Stack Overflow](https://stackoverflow.com/questions/71961921/specify-runner-to-be-used-depending-on-condition-in-a-github-actions-workflow/71973229#71973229) 

<details>
<summary>回避策を施したワークフローの例:</summary>
<div>

```yml
name: 09.choice-runner-os
on:
  workflow_dispatch:
    inputs:
      runs-on:
        description: 'OS'
        required: true
        defaults: "ubuntu-latest"
        type: choice
        options:
          - "ubuntu-latest"
          - "windows-latest"
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      runner: ${{ steps.step1.outputs.os }}
    steps:
    - name: 🎯 Determine OS
      id: step1
      run: |
        if [[ '${{ inputs.runs-on }}' == windows* ]]; then
          echo "os=windows-latest" >> $GITHUB_OUTPUT
        elif [[ '${{ inputs.runs-on }}' == macos* ]]; then
          echo "os=macos-latest" >> $GITHUB_OUTPUT
        else
          echo "os=ubuntu-latest" >> $GITHUB_OUTPUT
        fi
    - name: Determine OS結果 - '${{ steps.step1.outputs.os }}'
      run: echo "Determine OS結果 - '${{ steps.step1.outputs.os }}'"

  sample:
    needs: setup
    runs-on: ${{ needs.setup.outputs.runner }}
    steps:
    - name: おためし
      run: |
          echo "💡 This workflow on ${{ github.repository }} was started by ${{ github.actor }}"
          echo ""
          echo "📝 The runner context is:"
          echo "${{ toJson(runner) }}"
          echo ""
```

</details>



<!-- リンク -->
[jobs.<job_id>.runs-on]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on
