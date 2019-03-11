SRS EPP CLIENT (SEC)
====================

This is a simple command line interface to the EPP Proxy as part of the
NZ Shared Registry system.

It is designed to allow generation and transmission of all the XML actions
that the registry supports using command line arguments rather than having
to manually write the XML yourself.

Registrars may use that when integrating the SRS into their own systems
as well as for executing any one-off transactions that their system does
not support.

InternetNZ developers use the client when testing the SRS EPP Proxy.

Using `--dry-run` it is also possible to generate XML than can then be
sent using the `send-xml` action for any feature not supported by the
client itself.

REQUIRED MODULES
----------------

`sec` uses the following non-standard python modules: lxml, requests and
jinja2.

These are all available as Debian packages, to install them run:
`sudo apt-get install python3-lxml python3-requests python3-jinja2`.

ENVIRONMENT
-----------

The following environment variables can be used:

* `EPP_SERVER` Hostname of the EPP server. Used when `--server` is not
   specified.

* `EPP_PORT` Port the EPP server accepts connections on, defaults to 700.

* `EPP_CA_CERTIFICATE` SSL CA certificate that authenticates the HTTPS
   connection, used when `--ca-certificate` is not specified.

* `EPP_USERNAME` EPP client username (registrar ID), used when `--username`
   is not specified.

* `EPP_PASSWORD` EPP client password, used when `--password` is not specified.

* `EPP_CERTIFICATE` EPP client certificate signed by the registry, used
   when `--certificiate` is not specified.

* `EPP_KEY` EPP client private key, used when `--key` is not specified and
   will default to using the same file as `--certificiate`.

KEYS
----

The following keys are in the `registry/` directory:

* `srs-root-ca.pem` The SSL CA certificate used to authenticate the SSL
  connection.

USAGE
-----

Usage: `sec ACTION OPTIONS`. Run `sxc help` to see available
actions.

Each action has a help message that shows all the possible options for that
action and brief, what they do. Run `sxc help ACTION` to see that help, eg:

```sh
sec help domain-update
```

Most command line arguments are optional, you should always run the command
with `--dry-run` first to verify that the XML is what you want to send.

In the following examples, each option is shown on it's own line for readability,
your shell will require everything to be on one line.

An example that gets the server greeting.

```sh
sec hello
  --server srstest.srs.net.nz
  --certificate "./registrar-500.crt"
  --key "./registrar-500.key"
```

You can check that the username and password are correct using the `login`
command.

```sh
sec login
  --server srstest.srs.net.nz
  --username 500
  --password "test-password"
  --certificate "./registrar-500.crt"
  --key "./registrar-500.key"
```

The client password can be changed using `--new-password` for any action other
than `hello`. The easiest method is to use `login`.

```sh
sec login
  --server srstest.srs.net.nz
  --username 500
  --password "test-password"
  --new-password "new-test-password"
  --certificate "./registrar-500.crt"
  --key "./registrar-500.key"
```

All the following example assume that the client connection details have been
set using environment variables shown below.

In this case the client certificate and key are contained in a single file.

```sh
export EPP_SERVER="srstest.srs.net.nz"
export EPP_USERNAME="500"
export EPP_PASSWORD="test-password"
export EPP_CERTIFICATE="./registrar-500.pem"
```

Some options parsed by the client are context sensitive, they are sub-options
of the option that preceded it.

In this example, the sub-options to `--contact` are context specific.

```sh
sec contact-create
  --contact test-handle-1
    --name "Test"
    --email "test@example.co.nz"
    --street1 "Test Street1"
    --city "Test City"
    --country NZ
    --phone +64.40000123
```

And in this example, the sub-option to `--nameserver` are context specific.

```sh
sec domain-create
  --domain example.co.nz
  --period 12
  --registrant registrant-handle-1
  --admin admin-handle-1
  --nameserver ns1.example.co.nz
    --v4 192.168.0.1
  --nameserver ns2.example.net.nz
```

When updating a domain, details can be added or removed which is specified using
`--add` or `--remove`.

```sh
sec domain-update
  --domain example.co.nz
  --remove
    --admin admin-handle-1
    --nameserver ns1.example.co.nz
  --add
    --admin admin-handle-2
    --technical technical-handle-3
    --nameserver ns1.example.net.nz
```

The default update mode is `--add` so it can be omitted if the new details are
put first. The following is equivalent to the above.

```sh
sec domain-update
  --domain example.co.nz
  --admin admin-handle-2
  --technical technical-handle-3
  --nameserver ns1.example.co.nz
  --remove
    --admin admin-handle-1
    --nameserver ns1.example.co.nz
```

To send your own XML, use `send-xml` as the action, the client will authenticate
first before sending the request.

```sh
sec send-xml
  --file request.xml
```
