# Security

---

## Supply Chain Attacks

A supply chain attack compromises a legitimate package to deliver malicious code to anyone who installs it. They happen fast — one recent attack on a popular Python package lasted 46 minutes before being caught. Your defenses need to work before you notice anything is wrong.

### Dependency hygiene

**Pin your dependencies.** The single most effective defense. Unpinned specifiers like `litellm>=1.0` let the resolver install any version, including a compromised one. Exact versions like `litellm==1.80.0` don't.

**Use lock files.** Tools like `uv`, `poetry`, and `pip-compile` generate lock files that freeze every dependency — direct and transitive — to an exact version. If your lock file predates an attack window, you're safe regardless of what specifiers say.

* Commit your lock file to the repo. When Claude Code scaffolds a project with `pyproject.toml`, make sure the lock file goes in too.

**Don't blindly upgrade.** `pip install --upgrade` on everything at once is risky. Wait a day or two before upgrading to brand-new versions of popular packages — most supply chain attacks are caught quickly.

### Transitive dependencies

You don't have to `pip install` a package directly to be exposed. If anything in your environment depends on it, you're affected.

```bash
pip show <package>   # check if a package is installed and where it came from
pip list             # show everything installed in the environment
```

### Secrets and credentials

* Don't store API keys in project files. Use environment variables set outside your codebase.
* Rotate keys periodically.
* If you suspect you were affected by a compromised package, rotate **every** credential on the machine — not just the ones you think the attack targeted.

### The broader pattern

This applies to every ecosystem — Python, npm, Go modules, etc. The attack vector differs but the defense is the same: **pin versions, use lock files, and don't assume "latest" means "safe."**
