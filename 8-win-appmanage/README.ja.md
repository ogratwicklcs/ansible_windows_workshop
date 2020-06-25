# Exercise 8 - Windows Application Management  

Another big motivation for managing Windows with Ansible is automation of application management. 

There are several ways to manage applications on Windows with Ansible. For example, the win_feature module from Exercise 2 can be used to manage applications provided by the OS. On the other hand, if it is not a function provided by the OS but by a 3rd party application, there is a module called win_package and win_chocolatey.

### Preparation 

To use Chocolatey, you need to install chocolatey software on your Windows host. Let's also automate this with Ansible. Using Exercise 2 as a reference, use the PowerShell ad hoc commands win_shellto do the following:

In Ansible Tower, Inventory → Windows Workshop Inventory → Hosts

Check　the "student1-win1 " and click "Run Command" .

On the Run Command screen, select the options below.

**Module:**  
　win_shell

**Argument:**
　 Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

**Machine Credentials:**
　Student Account

Chocolatey is now available on your Windows host. Ready!!!

### Step 1:

![Student Playbooks](images/8-vscode-existing-folders.ja.jpg)

After clicking on README.md, hover over the WORKSHOP_PROJECT section and click the New Folder button.

Type `win_chocolatey` and hit enter. Now, click that folder so it is selected. 

Right-click the `win_chocolatey` folder and select New File.

Type `app_list.yml` and hit enter. 

Right-click the `win_chocolatey` folder and select New File.

Type `app_manage.yml` and hit enter.

![Empty site.yml](images/8-create-list-empty.ja.jpg)

## Creating a playbook

We are going to create two playbooks now.

1. app_list.yml:
　Display a list of applications managed via chocolatey.
2. app_manage.yml:
　Add, remove and update applications.

First, let's create from 1.
app_list.ymlMake sure that the editor for editing the Playbook is open in the right pane and do the following:

<!-- {% raw %} -->
```yaml
---
- hosts: windows
  name: This is my Windows application list playbook

  tasks:
  - name: Gather facts from chocolatey
    win_chocolatey_facts:

  - name: Displays the Packages
    debug:
      var: ansible_chocolatey.packages
```
<!-- {% endraw %} -->

![app_list.yml](images/8-create-list.ja.jpg)

> **Tips**
>
> `win_chocolatey_facts:` This module gets information about applications managed by chocolatey. This time, try running it before and after installing the application to check the differences in the installed packages.


Then、`app_manage.yml` click on and edit the playbook as follows:   

<!-- {% raw %} -->
```yaml
---
- hosts: windows
  name: his is my Windows application management playbook

  tasks:
  - name: Install Chrome
    win_chocolatey:
      name: googlechrome
      state: present
```
<!-- {% endraw %} -->

![app_manage.yml](images/8-create-mamage.ja.jpg)

> **Tips**
>
> `win_chocolatey:` A module that adds, deletes, and updates applications in cooperation with the chocolatey repository. This time we installed Googoe Chrome as an example. 

## Save and commit

The playbook that displays the application list with chocolatey and the playbook that manages the application are complete. Save your changes and commit to GitLab like in the previous exercises.  

## Creating a job template

Now that you have created a new playbook, go back to the Ansible Tower GUI and sync your project.
Next, you need to create a new job template to run this playbook. Go to Templates , click Add and select Job Template to create a new job template.

### 1. Creating a job template for app_list.yml

Fill out the form with the following values:  

| Key                | Value                      | Remarks |
|--------------------|----------------------------|------|
| name               |Windows アプリケーション取得           |      |
| Job type           | Execution                        |      |
| Inventory          | Windows Workshop Inventory |      |
| Project            | Ansible Workshop Project   |      |
| Playbook           | `chocolatey/app_list.yml`     |      |
| Credentials | Student Account            |      |
| Limit              | windows                    |      |
| Options            | 	[*] Check to enable fact cache      |      |

![Create Job Template](images/8-win_applist-template.ja.jpg)

保存をクリックします。  

### 2. app_manage.yml 用のジョブテンプレート作成  

同様にして、アプリケーション管理のジョブテンプレートを作成します。  

値は下記参照ください。    

| キー                | 値                      | 備考 |
|--------------------|----------------------------|------|
| 名前               |Windows アプリケーション管理|      |
| 説明        |                            |      |
| ジョブタイプ           | 実行                        |      |
| インベントリー          | Windows Workshop Inventory |      |
| プロジェクト            | Ansible Workshop Project   |      |
| PLAYBOOK           | `chocolatey/app_manage.yml`     |      |
| 認証情報 | Student Account            |      |
| 制限              | windows                    |      |
| オプション            | [*] ファクトキャッシュの有効化にチェック      |      |

![Create Job Template](images/8-win_appmanage-template.ja.jpg)

### Playbook の起動

作成した Playbook を以下の順番に実行してみて、表示内容を確認してみましょう♬  

1. `Windows アプリケーション一覧取得` ジョブテンプレートの起動
2. `Windows アプリケーション管理` ジョブテンプレートの起動
3. `Windows アプリケーション一覧取得` ジョブテンプレートの起動

どうなりましたでしょうか？Chromeは追加でインストールされましたか？  
インストールされると、以下のように表示されると思います。  

![Create Job Template](images/8-chrome-installed.ja.jpg)

今回は演習を簡略化するため変数は使いませんでしたが、アプリケーション管理のところで、state オプションを変数化し、present/absent/latest を選択できるようにしておくと追加、削除、更新が実行時に選択出来て便利かもしれません。また、アプリケーションの一覧をcsv化して、そのファイルを読み込んできて複数のアプリケーションを一気にインストールなどということももちろん可能です。便利ですね。♬  

これでオプションのハンズオンも終了です！お疲れさまでした！！  
