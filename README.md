# yaml-crypt

[![Build Status](https://img.shields.io/travis/pascalgn/yaml-crypt.svg?style=flat-square)](https://travis-ci.org/pascalgn/yaml-crypt) [![Coverage status](https://img.shields.io/coveralls/github/pascalgn/yaml-crypt.svg?style=flat-square)](https://coveralls.io/github/pascalgn/yaml-crypt) [![NPM version](https://img.shields.io/npm/v/yaml-crypt.svg?style=flat-square)](https://www.npmjs.org/package/yaml-crypt)

Command line utility to encrypt and decrypt YAML documents.

## Installation

The package is available on the [npm registry](https://www.npmjs.com/package/yaml-crypt), so just run

    $ yarn global add yaml-crypt
    $ yaml-crypt --help

You can also install the package locally:

    $ mkdir yaml-crypt && cd yaml-crypt
    $ yarn init --yes
    $ yarn add yaml-crypt
    $ ./node_modules/.bin/yaml-crypt --help

## Usage

First you will need to generate a key file. Currently, only the [Fernet](https://github.com/fernet/spec/blob/master/Spec.md)
encryption scheme is supported, so you will need a key with exactly 32 bytes.
The easiest way is to use the [pwgen](https://linux.die.net/man/1/pwgen) command:

    $ pwgen 32 1 > my-key

Another way would be to use the `urandom` device file:

    $ cat /dev/urandom | LC_ALL=C tr -dc A-Za-z0-9 | head -c 32 > my-key

To encrypt all values in a YAML file, run

    $ yaml-crypt -k my-key my-file.yaml

This will generate the file `my-file.yaml-crypt`, while leaving `my-file.yaml` intact.
If you want to delete the original file after encryption, use the `--rm` option.

> Files will be deleted using [unlink](https://linux.die.net/man/2/unlink).
> If this does not meet your security needs, consider removing the file manually instead!

The operation will be performed based on the file extension, so to decrypt a file,
just use

    $ yaml-crypt -k my-key my-file.yaml-crypt

You can also encrypt only certain parts of a file. Given the following YAML file

    apiVersion: v1
    kind: Secret
    data:
      username: user1
      password: secret123

you can use `--path data` to only encrypt the values `user1` and `secret123`.

Kubernetes secrets are Base64 encoded, so you should use the `--base64` option.

It is also possible to directly open encrypted files in an editor, decrypting them
before opening and encrypting again when saving:

    $ yaml-crypt -E my-file.yaml-crypt

## Configuration

The yaml-crypt command looks in `~/.yaml-crypt` for a file `config.yaml` or `config.yml`.
Currently, only the `keys` property is supported. These keys will be used when no key
files are given on the command line:

    $ cat ~/.yaml-crypt/config.yaml
    keys:
    - file: /home/user/.my-key-file
    - key: my-raw-key-data
    - key: !!binary my-base64-key-data
    $ yaml-crypt my-file.yaml

All whitespaces at the beginning and end of keys will be removed when reading keys.

## License

The yaml-crypt tool is licensed under the MIT License
