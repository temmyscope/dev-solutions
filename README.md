# dev-solutions
The intent of this repo is to provide answers and or solutions to commonly experienced problems/development decisions when installing development tools or setting up your dev environment


# Problem 1: Issues Installing Pycurl on MacOS

- Reference: https://gist.github.com/vidakDK/de86d751751b355ed3b26d69ecdbdb99

## Introduction
Installing `pycurl` on MacOS with Python 3.6+ and newer proved to be tricky, especially since a lot of the available resources seem to be outdated ([1], [2], [3]).

This gist was last updated at 2020-12-31.


## Steps:
1. Remove previously installed pycurl version and Homebrew versions of `curl`:
```bash
    pip uninstall pycurl
    brew uninstall curl
    brew uninstall openssl
    brew uninstall curl-openssl
```
2. Install `openssl` and a `curl` version that uses openssl rather than SecureTransport, which MacOS `curl` uses by default. Note that Homebrew no longer allows options while installing, so it's not possible to do `brew install curl --with-openssl`. This was replaced with a different brew formula, `curl-openssl`, and again replaced on `2020-01-20` with the current version of just `curl`.:
```sh
    brew install openssl
    brew install curl  # this is the latest version with openssl
```
3. Now we need to use the new curl when installing pycurl. We can also export the path to the new curl to make it used by default:
```bash
    echo 'export PATH="/usr/local/opt/openssl@3/bin:$PATH"' >> ~/.zshrc  # or ~/.bash_profile
    echo 'export PATH="/usr/local/opt/curl/bin:$PATH"' >> ~/.zshrc
    source ~/.zshrc
    source ~/.virtualenvs/foo_venv/bin/activate  # or whichever venv you're using
    export PYCURL_SSL_LIBRARY=openssl
    export LDFLAGS="-L/usr/local/opt/curl/lib"
    export CPPFLAGS="-I/usr/local/opt/curl/include"
    
    # Use the next two lines only if installation fails after the above steps,
    # Especially if the failure contains this: "src/pycurl.h:178:13: fatal error: 'openssl/ssl.h' file not found"
    export LDFLAGS="-L/usr/local/opt/openssl@3/lib"
    export CPPFLAGS="-I/usr/local/opt/openssl@3/include"
    
    pip install --no-cache-dir --compile --ignore-installed --config-setting=“--build-option=---with-openssl” pycurl
```
4. Test with:
```python
    python
    >>> import pycurl
```

## There should now be two different versions of curl:
1. The original OS X one in `/usr/bin/curl`:
```bash
    $ /usr/bin/curl --version
curl 7.64.1 (x86_64-apple-darwin20.0) libcurl/7.64.1 (SecureTransport) LibreSSL/2.8.3 zlib/1.2.11 nghttp2/1.41.0
Release-Date: 2019-03-27
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp
Features: AsynchDNS GSS-API HTTP2 HTTPS-proxy IPv6 Kerberos Largefile libz MultiSSL NTLM NTLM_WB SPNEGO SSL UnixSockets
```

2. The Homebrew one in `/usr/local/opt/curl/bin/curl`:
```bash
    $ /usr/local/opt/curl/bin/curl --version
curl 7.74.0 (x86_64-apple-darwin20.2.0) libcurl/7.74.0 (SecureTransport) OpenSSL/1.1.1i zlib/1.2.11 brotli/1.0.9 zstd/1.4.8 libidn2/2.3.0 libssh2/1.9.0 nghttp2/1.42.0 librtmp/2.3
Release-Date: 2020-12-09
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps mqtt pop3 pop3s rtmp rtsp scp sftp smb smbs smtp smtps telnet tftp
Features: alt-svc AsynchDNS brotli GSS-API HTTP2 HTTPS-proxy IDN IPv6 Kerberos Largefile libz Metalink MultiSSL NTLM NTLM_WB SPNEGO SSL TLS-SRP UnixSockets zstd
``` 

And our `pycurl` should use the second one, both at linker level and at compile level.

[1]: https://cscheng.info/2018/01/26/installing-pycurl-on-macos-high-sierra.html
[2]: https://github.com/transloadit/python-sdk/issues/4
[3]: https://www.itnota.com/curl-http2-macos/



# Problem 2: Next Problem
