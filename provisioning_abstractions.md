Simple abstractions for provisioning your infrastructure
========================================================

```bash
function _include {
  pushd $1
  source init.sh
  popd
}
```

```bash
function _package {
  local CMD="sudo apt-get --yes --quiet install $1"
  if [[ -z $2 ]]; then
    $CMD
  else
    $CMD=$2
  fi
}
```
