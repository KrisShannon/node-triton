# node-triton changelog

## 4.5.2 (not yet released)

(nothing yet)


## 4.5.1

- #92 `triton` CLI should summarize `err.body.errors` from CloudAPI
  Per <https://github.com/joyent/eng/blob/master/docs/index.md#error-handling>,
  CloudAPI error response will sometimes have extra error details to show.


## 4.5.0

- #88 'triton inst tag ...' for managing instance tags.


## 4.4.4

- #90 Update sshpk and smartdc-auth to attempt to deal with multiple package
  inter-deps.


## 4.4.3

- #86 Ensure `triton profile ls` and `triton profile set-current` work
  when there is no current profile.


## 4.4.2

- Support `triton.createClient(...)` creation without requiring a
  `configDir`. Basically this then fallsback to a `TritonApi` with the default
  config.


## 4.4.1

- #83, #84 Fix running `triton` on Windows.
  Note: Triton config is stored in "%APPDATA%/Joyent/Triton/..." on Windows,
  "~/.triton/..." on other platforms.


## 4.4.0

- #78 `triton image delete IMAGE`
- #79 Fix `triton instance get NAME` to make sure it gets the `dns_names` CNS
  field.
- PUBAPI-1227: Note that `triton image list` doesn't include Docker images, at
  least currently.


## 4.3.1

- #77 triton create error in v4.3.0


## 4.3.0

**Bad release. Use >=4.3.1.**

- #76 `triton image create ...` and `triton image wait ...`
- #72 want `triton image` to still return image details even when it is not in 'active' state


## 4.2.0

- Bash completion: Add completion for *args* to `triton account update <TAB>`.
  This isn't perfect because a space is added after completion of "FIELD=",
  but hopefully is helpful.
- #75 `triton account update ...`


## 4.1.0

- Unhide `triton completion` so hopefully more find it and use it.

- node-triton#73 `triton instance list --credentials` to include
  "metadata.credentials" in instance listing.

- node-triton#35 More easily distinguish KVM and LX and Docker images and
  instances.

    In PUBAPI-1161 CloudAPI (v8.0.0) started exposing IMG.type, INST.brand and
    INST.docker. One of the main issues for users is that telling KVM ubuntu
    from LX ubuntu is confusing (see also joyent/smartos-live#532).

    tl;dr:

    - `triton image list` default output now includes the `type` instead of
      `state`. The `state` column is still in output with `-l`, `-j`,
      `-o state`.
    - `triton instance list` default output now includes a `flags` column
      instead of `primaryIp`. The 'D' and 'K' flags identify Docker and KVM
      instances.
    - `triton instance list -l` includes the brand.

    Default output examples showing the various cases (and the attempt to
    stay within 80 columns):

    ```bash
    $ triton imgs
    SHORTID   NAME            VERSION   FLAGS  OS       TYPE          PUBDATE
    1bd84670  minimal-64-lts  14.4.2    P      smartos  zone-dataset  2015-05-28
    b67492c2  base-64-lts     14.4.2    P      smartos  zone-dataset  2015-05-28
    ffe82a0a  ubuntu-15.04    20151105  P      linux    lx-dataset    2015-11-05
    8a1dbc62  centos-6        20160111  P      linux    zvol          2016-01-11

    $ triton insts
    SHORTID   NAME         IMG                    STATE    FLAGS  AGE
    da7c6edd  cocky_noyce  3d996aaa               running  DF     10m
    deedeb42  ubu0         ubuntu-15.04@20151105  running  -      9m
    aa9ccfda  mini2        minimal-64-lts@14.4.2  running  -      9m
    e8fc0b96  centi0       centos-6@20160111      running  K      8m
    ```

- Filtering instances on `docker=true`:

    ```bash
    $ triton insts docker=true
    SHORTID   NAME         IMG       STATE    FLAGS  AGE
    da7c6edd  cocky_noyce  3d996aaa  running  DF     13m
    ```


## 4.0.1

- Add `triton env -t` to be able to emit a shell environment to configure `triton` itself.
  This allows one to have the following Bash function to select a Triton profile for
  `triton` and node-smartdc tooling:

        function triton-select { eval $(triton env $1); }


## 4.0.0

- [backwards incompat] #66 New consistent `triton` CLI style. See [the
  issue](https://github.com/joyent/node-triton/issues/66) for discussion.

    The major changes is that where some sub-commands used to be some
    flavour of:

        triton things       # list all the things
        triton thing ID     # get a thing
        triton thing -a ID  # create a new thing

    Now commands are consistently:

        triton thing list       # list all the things
        triton thing get ID     # get a thing
        triton thing create ... # create a new thing
        ...

    The most annoying incompatility is the need for "get" to
    get a thing. E.g.:

        BEFORE                  AFTER
        triton img blah         triton img get blah
        triton inst web0        triton inst get web0

    For *listing* things, there is typically a shortcut with
    the old form, e.g. `triton images` is a shortcut for
    `triton image list`.

    Currently all of the CLI *except* the experimental `triton rbac ...`
    is converted to the new consistent style.

- [backwards incompat] `triton whoami` was dropped. This used to be a shortcut
  for `triton account get`. It could possibly come back.

- *Much* improved [Bash
  completion](https://github.com/joyent/node-triton#bash-completion). See
  `triton completion -h` for notes on how to install.

- Add the ability to create a profile copying from an existing profile,
  via `triton profile create --copy NAME`.

- `triton key add` was added (<https://apidocs.joyent.com/cloudapi/#CreateKey>).


## 3.6.0

- #67 Add `triton create --network,-N NETWORK ...` option for specifying
  networks for instance creation. "NETWORK" is a network id, name, or
  short id; or a comma-separated array of networks.


## 3.5.0

- #67 Add `triton create --tag|-t ...` option for adding tags on instance creation.
  E.g. `triton create -n NAME -t foo=bar -t @my-tags-file.json IMAGE PACKAGE`.


## 3.4.2

- #63 "triton images" with a filter should not be cached.
- #65 Fix `triton profile(s)` handling when the user has no profiles yet.


## 3.4.1

- #60 Display `vcpus` in `triton packages` output.
- Add `-d,--data <data>` option to `triton cloudapi`.
- Fix `triton rbac role ROLE`. Also get that command to have a stable order for the
  displayed fields.


## 3.4.0

- Improvements for using node-triton as a module. E.g. a simple example:

        var triton = require('triton');
        var client = triton.createClient({profileName: 'env'});
        client.listImages(function (err, imgs) {
            console.log(err);
            console.log(imgs);
        });

  See the README and "lib/index.js" for more info.


## 3.3.0

- #59 CLI options to `triton create` to add metadata on instance creation:
    - `triton create -m,--metadata KEY=VALUE` to add a single value
    - `triton create -m,--metadata @FILE` to add values from a JSON
      or key/value-per-line file
    - `triton create -M,--metadata-file KEY=FILE` to set a key from a file
    - `triton create --script FILE` to set the special "user-script" key
      from a file


## 3.2.0

- #58 `triton --act-as=ACCOUNT ...` for an operator account to auth as
  themself, but operator on another account's resources. Note that operator
  accesses like this are audited on the CloudAPI server side.
- `triton --accept-version VER` hidden top-level option for development. This
  allows calling the target cloudapi with the given value for the
  "Accept-Version" header -- which is how CloudAPI does API versioning.
  By default `triton` is coded to a particular cloudapi version range, so
  forcing a different version *could* result in breaking in the triton client
  code that handles the response. IOW, this is just a tool for developers
  of this Triton client and CloudAPI itself.


## 3.1.0

- New (hidden for now, i.e. experimental) `triton env ...` to dump
  `eval`able shell commands for
  [node-smartdc](https://github.com/joyent/node-smartdc) environment setup for
  a given Triton CLI profile. E.g.:

        eval $(triton env east1)
        sdc-listmachines

  I think this should grow to support setting up Docker env as well.
- #54 `triton rbac role-tags` for now can't be hidden (as long we have the
  need to role-tag raw resource URLs like '/my/images').
- #54 `triton rbac apply --dev-create-keys-and-profiles` for
  experimenting/dev/testing to quickly generate and add user keys and setup
  Triton CLI profiles for all users in the RBAC config.
- #54 RBAC support, see <https://docs.joyent.com/public-cloud/rbac> to start.
    - `triton rbac info` improvements: better help, use brackets to show
      non-default roles.
    - `triton rbac reset`
    - change `triton rbac user USER` output a little for the 'keys' (show
      the key fingerprint and name instead of the key content), 'roles',
      and 'default_roles' fields.
- #54 *Drop* support for shortIds for `triton rbac {users,roles,policies}`
  commands. They all have unique *`name`* fields, just use that.
- #54 `triton rbac apply` will implicitly look for a user key file at
  "./rbac-user-keys/$login.pub" if no `keys` field is provided in the
  "rbac.json" config file.
- Change default `triton keys` and `triton rbac keys` output to be tabular.
  Otherwise it is a little obtuse to see fingerprints (which is what currently
  must be included in a profile). `triton [rbac] keys -A` can be used to
  get the old behaviour (just the key content, i.e. output appropriate
  for "~/.ssh/authorized\_keys").


## 3.0.0

- #54 RBAC support, see <https://docs.joyent.com/public-cloud/rbac> to start.
    - [Backward incompatible.] The `triton` CLI option for the cloudapi URL has
      changed from `--url,-u` to **`--url,-U`**.
    - Add `triton --user,-u USER` CLI option and `TRITON_USER` (or `SDC_USER`)
      environment variable support for specifying the RBAC user.
    - `triton profiles` now shows the optional `user` fields.
    - A (currently experimental and hidden) `triton rbac ...` command to
      house RBAC CLI functionality.
    - `triton rbac users` to list all users.
    - `triton rbac user ...` to show, create, edit and delete users.
    - `triton rbac roles` to list all roles.
    - `triton rbac role ...` to show, create, edit and delete roles.
    - `triton rbac policies` to list all policies.
    - `triton rbac policy ...` to show, create, edit and delete policies.
    - `triton rbac keys` to list all RBAC user SSH keys.
    - `triton rbac key ...` to show, create, edit and delete user keys.
    - `triton rbac {instance,image,network,package,}role-tags ...` to list
      and manage role tags on each of those resources.
    - `triton rbac info` will dump a summary of the full current RBAC
      state. This command is still in development.
    - `triton rbac apply` will synchronize a local RBAC config (by default it
      looks for "./rbac.json") to live RBAC state. Current the RBAC config
      file format is undocumented. See "examples/rbac-\*" for examples.
- #55 Update of smartdc-auth/sshpk deps, removal of duplicated code for
  composing Authorization headers


## 2.1.4

- #51: Update deps to get dtrace-provider 0.6 build fix for node v4.2.x.
- #49: `triton create ... --firewall` to enable [Cloud
  Firewall](https://docs.joyent.com/public-cloud/network/firewall).


## 2.1.3

- #44 'triton rm' alias for delete
- #43 `triton profile ...` doesn't use the profile from `TRITON_PROFILE` envvar

## 2.1.2

- #41 Add compatibility with ed25519 keys in ssh-agent
- #42 Tools using sshpk should lock in an exact version

## 2.1.1

- #40 Update smartdc-auth so that newer OpenSSH `ssh-keygen` default
  fingerprint formats for setting `keyId` work.
- #39 Test suite: Change the test config 'destructiveAllowed' var to
  'writeActionsAllowed'.


## 2.1.0

- Errors and exit status: Change `Usage` errors to always have an exit status
  of `2` (per common practice in at least some tooling). Add `ResourceNotFound`
  error for `triton {instance,package,image,network}` with exit status `3`.
  This can help tooling (e.g. the test suite uses this in one place). Add
  `triton help` docs on exit status.

- Test suite: Integration tests always require a config file
  (either `$TRITON_TEST_CONFIG` path or "test/config.json").
  Drop the other `TRITON_TEST_*` envvars.


## 2.0.0

- Changed name to `triton` npm package, graciously given up by
  [suguru](https://www.npmjs.com/~suguru) from his
  <https://github.com/ameba-proteus/node-triton> project. <3
  The latest previous release of the triton package was 1.0.7,
  so we'll separate with a major version bump for *this* triton
  package.

## 1.0.0

Initial release as `joyent-triton` npm package.
