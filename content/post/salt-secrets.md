+++
date = "2018-02-07T00:00:00+00:00"
draft = false
title = "Protecting Secrets in SaltStack Configuration"
+++

I came across a situation recently where secrets were being leaked via [SaltStack](https://docs.saltstack.com/en/latest/) deployments, run from a CI job.  I'll explain how this can happen, and give some suggestions to reduce the risk.

A common workflow in SaltStack is to store secrets in a [pillar](https://docs.saltstack.com/en/latest/topics/tutorials/pillar.html) on the salt master, which allows data to be [encrypted](https://docs.saltstack.com/en/latest/topics/pillar/index.html#pillar-encryption) until compilation.  The [`file.managed`](https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html#salt.states.file.managed) module can then be used to templatize configuration files and inject secrets from the encrypted pillars.

The problem appears when the file templates are changed within the salt state; by default, `file.managed` will display a diff of the file changes, and any secrets that would be injected into the file will be shown in cleartext in the diff.

## Example

Suppose we have a salt pillar that contains an encrypted password (the plaintext is `my57r0nkP455w0rd!`):

```yaml
app:
    password: |
        -----BEGIN PGP MESSAGE-----
            Version: GnuPG v1

            hQEMA/yXXF3hJbt6AQf+JCGag2mOyVmIi7QNRF2eagUtktot8sujCDVTCOTJe6tM
            5+bxlJUaqK2iZA7sO6arMu8myZTor9A0gDf+s+ij/S4LAURR4CT2OX/wI3gpnQw0
            oJt2bztbcl7ZqsRM1DP943Rx5XPOksCFvFR1ftzfF9zoohkpS9axx2WyQWnQqCs0
            x3QOr9RpBoMZU153N2YnG8ys5tskFQMHMaGGhXJSln8vDw9hodWTzO0OxS67Wp5L
            3fty7ruPZiJ/jeoKjC5BLPl0IFOJ0/ctRUF70OgEy72UozX7TD6sGghUwF16aDcc
            EZf9CUZHed8GMVOGAU5ZoDCFwSrF3ZGK6L8Ens6269JMARp2jf/+nI7/qCtVP+qy
            XGFur7xY/XkmMIhXcqeaBT6ZZojCMBgFOv0A2P7zFdl1uIFLeA+Uk/DfZhQLNxa/
            7y96iQXPL4CZVXUuAw==
            =bIm3
            -----END PGP MESSAGE-----
```

We want our state to copy a simple, YAML config file onto the root of our target systems; the file template looks like this:

```yaml
password: {{ pillar['app']['password'] }}
```

And the corresponding salt state, `config-example` that looks like:

```yaml
copy yaml config:
    file.managed:
        - name: /config.yml
        - source: salt://config-example/config.yml
        - template: jinja
```

When we apply the salt state, we get output like this:

```bash
$ sudo salt 'test-machine' state.apply config-example
test-machine:
----------
          ID: copy yaml config
    Function: file.managed
        Name: /config.yml
      Result: True
     Comment: File /config.yml updated
     Started: 00:57:23.942488
    Duration: 43.845 ms
     Changes:
              ----------
              diff:
                  New file
              mode:
                  0644

Summary for test-machine
------------------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  43.845 ms
```

Now suppose we add another parameter to the config template:

```yaml
password: {{ pillar['app']['password'] }}
new_parameter: true
```

When we reapply the state, we see a diff (and our password!):

```bash
$ sudo salt 'test-machine' state.apply config-example
test-machine:
----------
          ID: copy yaml config
    Function: file.managed
        Name: /config.yml
      Result: True
     Comment: File /config.yml updated
     Started: 01:09:38.271871
    Duration: 35.597 ms
     Changes:
              ----------
              diff:
                  ---
                  +++
                  @@ -1 +1,2 @@
                   password: my57r0nkP455w0rd!
                  +new_parameter: true

Summary for test-machine
------------------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  35.597 ms
```

Depending on how the salt state is applied, this output can end up any number of places; for example, when run from a Jenkins job the output above will land on filesystem of the build master, with the password in plain text.  Depending on how the retention policies for build logs are set, these credentials can accumulate over time.

## Solutions

There are a couple of ways to address this issue--locally in `file` module itself, and globally in the salt configuration files.

### Local Settings

The `file.managed` function has a boolean parameter called `show_changes`; by default its value is `True`, but setting it to `False` prevents the diff from being displayed:

```bash
$ sudo salt 'test-machine' state.apply config-example
test-machine:
----------
          ID: copy yaml config
    Function: file.managed
        Name: /config.yml
      Result: True
     Comment: File /config.yml updated
     Started: 01:09:38.271871
    Duration: 35.597 ms
     Changes:
              ----------
              diff:
                  <show_changes=False>

Summary for test-machine
------------------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  35.597 ms
```

### Global Settings

While it may be possible to enforce the use of the `show_changes` flag via git hooks or  similar, it's not always the best solution, and there may be many `file.managed` calls within a typical machine configuration.  Another way to control the output is via the [configuration](https://docs.saltstack.com/en/latest/ref/configuration/master.html) of the salt master itself  There are a few settings that govern output verbosity, but [`state_output`](https://docs.saltstack.com/en/latest/ref/configuration/master.html#state-output) is the one we're interested in.

If we set the `state_output` config value to `terse`, we'll get a simple success or failure message for our states, regardless of whether the `show_changes` flag is set on our state:

```bash
$ sudo salt 'test-machine' state.apply config-example
test-machine:
  Name: /config.yml - Function: file.managed - Result: Changed Started: - 01:41:00.549403 Duration: 35.582 ms

Summary for test-machine
------------------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  35.582 ms
```

If you need more verbose output while troubleshooting a salt state, you can always adjust the level at the command line, using `--state_output=full`.

## Conclusion

Even with gpg encryption, it's possible for plaintext secrets to wind up in the output of salt commands.  Pay attention to your output, and adjust your salt configuration to minimize exposure.
