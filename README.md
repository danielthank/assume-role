This tool will request and set temporary credentials in your shell environment variables for a given role.

## Installation

On OS X, the best way to get it is to use homebrew:

```bash
brew install danielthank/tap/assume-role
```

If you have a working Go environment:

```bash
$ go get -u github.com/danielthank/assume-role
```

On Windows with PowerShell, you can use [scoop.sh](http://scoop.sh/)

```cmd
$ scoop bucket add extras
$ scoop install assume-role
```

## Configuration

Setup a profile for each role you would like to assume in `~/.aws/config`.

For example:

`~/.aws/config`:

```ini
[profile chin.yenru]
region = ap-northeast-1
output = json

[profile stage]
# Stage AWS Account.
region = ap-northeast-1
role_arn = arn:aws:iam::1234:role/DeveloperAdministrator
mfa_serial = arn:aws:iam::9012:mfa/chin.yenru
source_profile = chin.yenru

[profile prod]
# Production AWS Account.
region = ap-northeast-1
role_arn = arn:aws:iam::5678:role/DeveloperAdministrator
mfa_serial = arn:aws:iam::9012:mfa/chin.yenru
source_profile = chin.yenru
```

`~/.aws/credentials`:

```ini
[chin.yenru]
aws_access_key_id = AKIMYFAKEEXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/MYxFAKEYEXAMPLEKEY
```

Reference: https://docs.aws.amazon.com/cli/latest/userguide/cli-roles.html

In this example, we have three AWS Account profiles:

 * chin.yenru
 * stage
 * prod

Each member of the org has their own IAM user and access/secret key for the `chin.yenru` AWS Account.
The keys are stored in the `~/.aws/credentials` file.

The `stage` and `prod` AWS Accounts have an IAM role named `DeveloperAdministrator`.
The `assume-role` tool helps a user authenticate (using their keys) and then assume the privilege of the `DeveloperAdministrator` role, even across AWS accounts!

## Usage

`assume-role` will output the temporary security credentials:

```bash
$ assume-role -r stage
export AWS_ACCESS_KEY_ID="ASIAI....UOCA"
export AWS_SECRET_ACCESS_KEY="DuH...G1d"
export AWS_SESSION_TOKEN="AQ...1BQ=="
export AWS_SECURITY_TOKEN="AQ...1BQ=="
export ASSUMED_ROLE="prod"
# Run this to configure your shell:
# eval $(assume-role prod)
```

If the role requires MFA, you will be asked for the token first:

```bash
$ assume-role -r stage
MFA code: 123456
```

Or windows PowerShell:
```cmd
$env:AWS_ACCESS_KEY_ID="ASIAI....UOCA"
$env:AWS_SECRET_ACCESS_KEY="DuH...G1d"
$env:AWS_SESSION_TOKEN="AQ...1BQ=="
$env:AWS_SECURITY_TOKEN="AQ...1BQ=="
$env:ASSUMED_ROLE="prod"
# Run this to configure your shell:
# assume-role.exe prod | Invoke-Expression
```

If you use `eval $(assume-role)` frequently, you may want to create a alias for it:

* zsh
```shell
alias assume-role='function(){eval $(command assume-role $@);}'
```
* bash
```shell
function assume-role { eval $( $(which assume-role) $@); }
```

Check `assume-role -h` for other usage
```cmd
Usage:
  assume-role [flags]

Flags:
  -d, --duration duration   The duration that the credentials will be valid for (default 1h0m0s)
  -h, --help                help for assume-role
  -o, --output string       Output format (default "bash")
  -r, --role string         Role to be switched to
```
