### 上传代码到github时, 用户名密码的方式无法授权 , 怎么办??

原因: github将之前的用户名密码授权的方式改成了token授权



github的授权方式由用户名密码变更为token的形式授权

https://blog.csdn.net/u012855229/article/details/117456925



### 解决

```shell
$ git push # 下面让你输入用户名和密码, 但是输入正确也无法授权
# 输入用户名
# 输入密码
# 无法通过

# https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/




# 解决步骤: 

# 1. 安装gh
$ brew install gh # 安装gh

# 2. 到github.com 上申请 Personal Access Token

# 3. 使用 gh 授权登录 (这步要用到步骤2中申请的那个token)
$ gh auth login # 用gh授权登录
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Paste an authentication token
Tip: you can generate a Personal Access Token here https://github.com/settings/tokens
The minimum required scopes are 'repo', 'read:org', 'workflow'.
? Paste your authentication token: ****************************************
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as phoenix19900219


# 4. push 代码到 github
$ git push # OK
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 2.41 KiB | 246.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/phoenix19900219/blogs.git
   b2553e3..8c17c7f  main -> main
```

