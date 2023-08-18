# Table of Contents

<!--ts-->

- [Add dependency in go mod in self-hosted gitlab](#add-dependency-in-go-mod-in-self-hosted-gitlab)
  - [Background](#background)
  - [How go get works?](#how-go-get-works)
  - [Fix](#fix)
    - [Use nginx](#use-nginx)
    - [Use caddy](#use-caddy)
  - [Config ssh](#config-ssh)

<!--te-->

# Add dependency in go mod in self-hosted gitlab

## Background

As a infrastructure engineer, it is common to deploy source code management tools like GitLab to management source code. When using GitLab for code management, we need to add dependencies to our projects like this:

```bash
go get -u -v gitlab.mycompany.com/mygroup/myproject
```

If we're using github, running `go get github.com/username/repo` will succeed. Here is an example:

```bash
go get -u -v github.com/lichuan6/go-mod-test
```

Output:

```
go: downloading github.com/lichuan6/go-mod-test v1.0.1
go: added github.com/lichuan6/go-mod-test v1.0.1
```

However, it fails with self-hosted VCS like gitlab:

```bash
go get gitlab.mycompany.com/mygroup/myproject
```

Output:

```
go: unrecognized import path "gitlab.mycompany.com/mygroup/myproject": parse https://gitlab.mycompany.com/mygroup/myproject?go-get=1: no go-import meta tags (meta tag gitlab.mycompany.com:443/mygroup/myproject did not match import path gitlab.mycompany.com/mygroup/myproject)
```

What does this error message mean?

## How go get works?

Before digging into the reason why it failds, we need to understand how `go get` works.

The [gitlab document](https://docs.gitlab.com/ee/development/go_guide/dependencies.html) says that prior to Go 1.12, the process for fetching a package was as follows:

- Query `https://{package name}?go-get=1`.
- Scan the response for the `go-import` meta tag.
- Fetch the repository indicated by the meta tag using the indicated VCS.

The meta tag should have the form `<meta name="go-import" content="{prefix} {vcs} {url}">`. For example, `gitlab.com/my/project git https://gitlab.com/my/project.git` indicates that packages beginning with `gitlab.com/my/project` should be fetched from `https://gitlab.com/my/project.git` using Git.

Looking back the error message, we know that we do not config correct meta tag for request like `https://{package name}?go-get=1`, here it refers to `https://gitlab.mycompany.com/mygroup/myproject`.

## Fix

The fix is easy, but it maybe different depending on which reverse proxy you use.

### Use nginx

If you're using `nginx`, you can add the following config to nginx configuration:

```
server {
    server_name gitlab.mycompany.com;

    if ($args ~* "^go-get=1") {
      set $condition goget;
    }
    if ($uri ~ ^/([a-zA-Z0-9\._-]+)/([a-zA-Z0-9_-]+)/?.*$) {
      set $condition "${condition}query";
    }
    if ($condition = gogetquery) {
      return 200 "<!DOCTYPE html><html><head><meta content='gitlab.mycompany.com/$1/$2 git https://gitlab.mycompany.com
/$1/$2' name='go-import'></head></html>";
    }
}
```

### Use caddy

If you're using `caddy`, you can add the following config to `Caddyfile`:

```
gitlab.mycompany.com {
        @goget {
                path_regexp goget ^/([a-zA-Z0-9\._-]+)/([a-zA-Z0-9_-]+)$
                query go-get=1
        }

        respond @goget 200 {
                body "<!DOCTYPE html><html><head><meta content='gitlab.mycompany.com/{re.goget.1}/{re.goget.2} git ssh://git@gitlab.mycompany.com/{re.goget.1}/{re.goget.2}' name='go-import'></head></html>"
        }

        reverse_proxy localhost:10080
}
```

## Config ssh

Now we have reverse proxy configured to return the correct meta tag in response, but there's still an error when running `go get -u -v gitlab.mycompany.com/mygroup/myproject`:

```
get "gitlab.mycompany.com/mygroup/myproject": found meta tag vcs.metaImport{Prefix:"gitlab.mycompany.com/mygroup/myproject", VCS:"git", RepoRoot:"ssh://git@gitlab.mycompany.com/mygroup/myproject"} at //gitlab.mycompany.com/mygroup/myproject?go-get=1
go: module gitlab.mycompany.com/mygroup/myproject: git ls-remote -q origin in /Users/username/go/pkg/mod/cache/vcs/d831e72abb0cb1006afb601471112319e0a1d0e490efb16e63a5682d76677a2e: exit status 128:
        Host key verification failed.
        fatal: Could not read from remote repository.

        Please make sure you have the correct access rights
        and the repository exists.
```

You need to add the following to ssh config file: `~/.ssh/config`, which means it wil tell go get to use the correct identity file to fetch the package from gitlab:

```
Host gitlab.mycompany.com
    # ssh://git@gitlab.mycompany.com/username/project.git
    Port 10022
    User git
    #IdentityFile ~/.ssh/id_rsa.gitlab.self-hosted
    IdentityFile ~/.ssh/id_rsa-gitlab.self-hosted
```

After all is setup, you can add dependency using `go get` successfully.

```
get "gitlab.tencet.com/mygroup/myproject": found meta tag vcs.metaImport{Prefix:"gitlab.mycompany.com/mygroup/myproject", VCS:"git", RepoRoot:"ssh://git@gitlab.mycompany.com/mygroup/myproject"} at //gitlab.mycompany.com/mygroup/myproject?go-get=1
go: added gitlab.mycompany.com/mygroup/myproject v0.0.1
```

🎉🎉🎉
