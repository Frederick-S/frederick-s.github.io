title: VS Code error - No source control providers registered
categories:
- Visual Studio Code
---

I opened a git repo by VS Code, but it shows `No source control providers registered`. One possible reason is that VS Code couldn't find the executable git command, so we can set the executable path of git in settings. Open `File -> Preferences -> Settings`, then find the `Git: Path` setting, click `Edit in settings.json`, and add `"git.path": "full-path-to-git"` to `settings.json`.

Reference:

- [Git missing in VS Code – No source control providers](https://stackoverflow.com/questions/46609255/git-missing-in-vs-code-no-source-control-providers)
