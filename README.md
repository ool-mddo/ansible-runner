---
# 使い方
ansible-builder version 3<br>
ansible-core 2.14.4
(最新バージョンの対応はTBD)

## mddo-ansible-runnerのファイル・ディレクトリ説明
- requirements.yml<br>
  ansible-galaxyにて導入するansible moduleを定義
- requirements.txt<br>
  ansibleのmoduleなどを使う上で必要なpipのライブラリーを定義
- execution-environment.yml<br>
  ansible-runnerのビルド環境を定義するyamlファイル
  詳細定義は下記を参照してください
  https://ansible.readthedocs.io/projects/builder/en/latest/definition/
- context/Dockerfile<br>
  ansible-builder createにて自動生成されるDockerfile
- context/configs/ansible.cfg<br>
  ansible-runnerコンテナ内に組み込むansibleの設定ファイルのansible.cfgを格納しておく
- .github/workflows/actions.yml<br>
  GithubActionsにてansible-runnerのコンテナイメージをビルドするActionsの定義

## Github ActionsにてAnsible-Runnerのイメージビルド
### ansible-builderを利用してDockerfileを生成する
```
git clone https://github.com/ool-mddo/mddo-ansible-runner.git
cd mddo-ansible-runner
ansible-builder create
```
※execution-environment.ymlを下記URLから最新の定義をペーストしておくことを推奨する<br>
　https://ansible.readthedocs.io/projects/builder/en/latest/definition/<br>
※仕様が変わってエラーメッセージが出た場合は都度エラーメッセージに応じた対応を実施する

### 生成されたDockerfileをもとにビルドを行う。
```
docker build -f context/Dockerfile -t ansible-execution-env:latest context
```

## Ansible-Runnerの使い方
### 参考ディレクトリの紹介
```
git clone https://github.com/ool-mddo/playground.git
cd playground/demo/copy_to_emulated_env
```
- inventory/hosts<br>
  ansible-runnerの実行するインベントリファイルを格納<br>
　※inventoryの内容は下記を参照<br>
　https://github.com/ool-mddo/playground/blob/main/demo/copy_to_emulated_env/inventory/hosts
- env
  - settings
    ansible-runnerの実行するときの設定を記述する<br>
    下の例はansible-runnerをdockerで実行するように指定する設定。
    ```
    process_isolation_executable: docker
    process_isolation: true
    ```
  - passwords
    ansible-runnerがSSHログインする先のサーバのログインパスワードおよびRoot権限になるためのパスワード情報を指定する<br>
    ※SSHやBECOMEのメッセージが下記例と違う場合は適宜メッセージ内容を修正する
    ```
    ---
    "^SSH password:\\s*?$": "login password"
    "^BECOME password.*:\\s*?$": "sudo password"
    ```
   - project/playbooks/
     上記パス配下にplaybookを格納する

ansible-runnerの起動例
```
ansible-runner run . -p /data/project/playbooks/test.yaml
```
使えるOption
- --container-option="--net=${API_BRIDGE}" <br>
  ansible-runnerを実行する際のansible-runnerの動作設定を指定できる<br>
  上記例はansible-runnerのコンテナを起動する際につながる仮想ブリッジ名を指定している
- --cmdline "-e login_user=${LOCALSERVER_USER}"<br>
  ansible-runnerを実行する際に、playbookへ渡す引数を指定できる。
