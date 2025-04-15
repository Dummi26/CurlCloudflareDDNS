# CurlCloudflareDDNS

A simple way to update cloudflare DNS records automatically using just `sh` and `curl`

Requires `sh`, `shuf` and `curl` in `$PATH`, DNS records need to exist.

## Configuring

For each DNS record, create a directory here.
In this directory, the following files are required:

`auth`:

Defines headers which are sent to cloudflare
when you want to change a DNS record.

```
Authorization: Bearer <an api token>
```

or:

```
X-Auth-Email: <email>
X-Auth-Key: <api key>
```

`check`:

A `sh` script which is called to check if your IP has changed.
The first argument, `$1`, is a random number.
The output from running `check` is split at whitespace (word splitting),
and all words in the output must be equal to the random number `$1`. If
this is not the case, it is assumed that the IP has changed.
If `check` exits with a nonzero exit code,
it is also assumed that the IP has changed.

```
echo "$1" > /tmp/curl_cloudflare_ddns_rand_num/example
curl "http://<domain>/curl_cloudflare_ddns_rand_num/example" || exit 1
```

`ip`:

A `sh` script which is called to get this machine's current IP,
to be passed to `set_to`.
It should output the current IP and nothing else.
Note: replace `-4` with `-6` for IPv6

```
curl -4 https://ipinfo.io/ip || exit 1
```

`settings`:

This file is sourced by `set_to` and defines the DNS record (except for
the IP, which is passed to set_to as its only argument).

```
export ZONE_ID=<cloudflare zone id>
export DNS_RECORD_ID=<cloudflare dns record id>
export RECORD_NAME=<domain.tld / sub.domain.tld>
export RECORD_PROXIED=true
export RECORD_TTL=120
export RECORD_TYPE=A
```

## Using

```
cd <configured directory>
sh ../run
```

This will start by `check`ing if your ip has changed.
If it has, it will get your new `ip` and run `set_to`
to update the DNS record.

You can check for IP changes regularly, for example using systemd.
Assuming your `check` works well enough, you will only
talk to your own server unless your IP has actually changed.
