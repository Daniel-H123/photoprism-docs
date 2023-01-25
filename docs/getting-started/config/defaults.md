# `defaults.yml`

Default values, including the *config* path to use, may optionally be specified in a `defaults.yml` file located in `/etc/photoprism`. Write permissions are not required, since all changes will be stored in the `options.yml` file.

With the variable `PHOTOPRISM_DEFAULTS_YAML` or the flag `--defaults-yaml`, you can also specify a custom defaults file to be loaded when PhotoPrism starts.

The [default values](index.md#authentication) can be overridden by values in an [`options.yml`](index.md) file, command flags, and environment variables. As with all global [config options](../config-options.md), changes require a restart to take effect.

!!! tldr ""
    When [editing YAML files](../../developer-guide/technologies/yaml.md), it is important that related values start at the same indentation level and that spaces are used, since tabs are not allowed. You can use any text editor for this.

### Example

Since you only need to specify the values for which you want to set a default, a `defaults.yml` file does not need to contain all possible options and could look like this, for example:

```yaml
Debug: false
ReadOnly: true
JpegQuality: 85
TrustedProxies:
  - "172.16.0.0/12"
  - "10.0.0.0/8"
```

### Config Options

↪ see [`options.yml`](index.md#authentication)
