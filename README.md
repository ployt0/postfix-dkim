[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fployt0%2Fpostfix-dkim.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fployt0%2Fpostfix-dkim?ref=badge_shield)

This repo is intended to be built from the Dockerfile by the end user.

Its objective is to be a minimal working example (MWE) of a usable MTA. Minimal so I don't undo any good work the postfix team has put into sensible defaults. Minimal in the hope it can be easily acknowledged as trustworthy and serve the purpose of signing outgoing mail using DKIM.

Aside from the usual; `docker build -t mydkimimg .`, `docker run`, and `docker exec`, there are 2 more requirements:

1. Change `ARG DOMAIN_NAME=example.com` to whatever domain name you will be using.
2. There are two options for providing a private key before starting a container:
    - Replace the private key, for which I've provided an example beginning `RUN echo "-----BEGIN PRIVATE KEY-----\n\` with your private key.
    - Volume mount, as `/etc/dkimkeys`, a directory containing your private key we are calling `dkim-key.pem`. For example:
    ```shell
    docker run -d --name dkimtest -v /etc/dkimkeys/forcontainer:/etc/dkimkeys mydkimimg
    ```
3. To provide a key after starting would require some kind of reloading which might simply be restarting the container. This option has been ignored in the interests of keeping things simple.

More details about the development and debugging of this image can be found [here](https://silverbullets/ci-cd/containerising-postfix-with-opendkim).

## Test

Please install `opendkim-tools`. Test using, eg:

```shell
opendkim-testkey -k /etc/opendkim/keys/silverbullets.co.uk/x.private -s x -d silverbullets.co.uk -vvv
```

Check `$?` for return code. `-vvv` can reveal errors even with a zero return code. I found a `key not secure` on someone else's repo/image.

It's a cool tool for checking your own key is right. Key management, by its repetitive nature, is rather prone to careless human error.

## Non-test

Don't be alarmed that the DKIM service isn't running:

```shell
root@silverbullets:/# service opendkim status
opendkim is not running ... failed!
```

This doesn't impact the other services. I can still send emails which "PASS" Gmail's **DKIM authentication** check.

## Demo

This is version 0.1, before TLS.

<video src="https://user-images.githubusercontent.com/25666053/196518942-1ad4d7bc-0560-4ef4-8556-25fc5314f236.mp4"></video>


## Receiving mail

When I first tagged this, v0.1, I never imagined it would serve equally well in receiving email. 

By default, `docker run` doesn't open any ports and you can experiment, in relative comfort, sending without listening. Check this with `docker ps`.

*All* we must do in order to receive mail with this image is open port 25.

```shell
:~$ docker run -d --name mailTA -p 25:25 -v /etc/dkimkeys/forcontainer:/etc/dkimkeys mydkimimg
:~$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                                                      NAMES
72b1c36e93a5   mydkimimg             "/bin/sh -c 'service…"   8 minutes ago   Up 8 minutes   0.0.0.0:25->25/tcp, :::25->25/tcp                                          mailTA
 ```

Congratulations. I've assessed the security of receiving email at [https://testconnectivity.microsoft.com/tests/exchange](https://testconnectivity.microsoft.com/tests/exchange) and it said I was not vulnerable. It tested open relay attacks and spoofing.

I don't know how that assessment missed that I am using self-signed keys for TLS. The keys don't even have the common name (CN) for my domain. I'll get around to fixing all this at some point. Just know that it *works*.

## Next steps

Opening up to receiving from the world isn't without risks. SpamAssassin is the software to mitigate these and it is a bit slow to install, so I'm working on a new repo which will build an image which will need personalisation once running. postfix-dkim will stay as just that, minimal and simple, suitable for outgoing only mail, which is a diminishing market as I know outlook want to email my MTA before accepting mail from it.
