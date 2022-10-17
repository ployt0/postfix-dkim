This repo is intended to be built from the Dockerfile by the end user.

It's objective is to be as simple as possible, in the hopes it can be easily acknowledged as trustworthy and serve the purpose of signing outgoing mail using DKIM.

There are two things required from the user apart from being able to type `docker build -t mydkimimg .`, `docker run` and `docker exec`:

1. Change `ARG DOMAIN_NAME=example.com` to whatever domain name you will be using.
2. There are two options for providing a private key:
    - Replace the private key, for which I've provided an example beginning `RUN echo "-----BEGIN PRIVATE KEY-----\n\` with your private key.
3. Volume mount, as `/etc/dkimkeys`, a directory containing your private key we are calling `dkim-key.pem`. For example:
    ```shell
    docker run -tid --name dkimtest -v /etc/dkimkeys/forcontainer:/etc/dkimkeys mydkimimg
    ```

More details about the development and debugging of this image can be found [here](https://silverbullets/ci-cd/containerising-postfix-with-opendkim).

## Test

Please install `opendkim-tools`. Test using, eg:

```shell
opendkim-testkey -k /etc/opendkim/keys/silverbullets.co.uk/x.private -s x -d silverbullets.co.uk -vvv
```

Check `$?` for return code. `-vvv` can reveal errors even with a zero return code. I found a `key not secure` on someone else's repo/image.

It's a cool tool for checking your own key is right. Key management, by it's repetitive nature, is rather prone to careless human error.

## Demo

<video src="https://user-images.githubusercontent.com/25666053/196518942-1ad4d7bc-0560-4ef4-8556-25fc5314f236.mp4"></video>
