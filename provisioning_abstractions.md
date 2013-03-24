Simple abstractions for provisioning your architecture
======================================================

```bash
function _include {
  pushd $1
  source init.sh
  popd
}
```

```bash
function _package {
  sudo apt-get install --yes --quiet install $1
}
```
