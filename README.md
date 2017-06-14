# one-liners

> Bash one-liners that are always useful :heart: :heart:

### Why?

There are [many](https://github.com/jlevy/the-art-of-command-line#basics)
[oneliners](https://github.com/stephenturner/oneliners)
[repositories](https://github.com/congto/oneliners) that more or less have a
ton of oneliners. This is a list of the commands that I use the most.

### `init`

#### Concepts

> These will help in building other commands

- Generate a sequence

    ```sh
    seq 1 100
    ```

- Run a command N times, with a sleep of T seconds in between
    ```sh
    for i in `seq 1 N`; do
      command;
      sleep T;
    done;
    ```


#### Commands

> This is the real deal! :fire:

- Run `hub ci-status` for a 100 times with a sleep of 2 seconds, right after `git push`.

    ```sh
    git push origin master && for i in `seq 1 100`; do hub ci-status; sleep 2; done;
    ```

- Git clone a really huge repository, using a shallow clone which is deepened in stages
    ```sh
    git clone --depth 1 REMOTE_URL folder;
    cd folder;
    for i in `seq 1 100`; do git fetch --depth=$i; done;
    ```

- SFTP `put` a file into a remote server, in one single command (SFTP shell doesn't have TAB autocompletion)

    ```sh
    sftp -P $PORT$ $USERNAME$@$SERVER$ <<< 'put $FILENAME$'
    ```

- Check what certificate an APK file has been signed with

    ```sh
    jarsigner -verify -verbose -certs apk-name.apk | less
    ```

- List all the keys in a keystore file (Android)

    ```sh
    keytool -list -v -keystore $FILENAME$.jks
    ```

- Replace target regular expression with replacement (using captured groups) with `sed`

    ```sh
    sed -ie "s/expressionWith\([0-9]*\)AGroup/expressionWithAGroup\1InADifferentPlace/g" *.xml
    ```

    **NOTE:** `sed` uses Basic Regular Expressions, hence the group capturing parenthese MUST be
    escaped: [SO Answer](http://stackoverflow.com/a/24717687/2080089)


- Curl command to run commands through a SOCKS proxy (setup either through TOR
    or ssh tunneling)

    ```sh
    curl --socks5 localhost:9050 http://icanhazip.com
    ```

    ```sh
    curl --socks5 localhost:9050 https://check.torproject.org > index.html
    $EDITOR index.html
    ```

    **NOTE:** This is especially useful when trying to figure out whether the
    socks5 proxy is working properly.

- Iterating over files inside `bash`

    ```sh
    for i in *.caf; do
    ffmpeg -i "$i" "$(echo "$i" | cut -d . -f 1)".wav
    done
    ```

    **Note:** Check [this](http://unix.stackexchange.com/a/131767/36994) Unix
    StackExchange answer for more details about whitespaces, quoting and bash.


- List the packages that are installed on your system with `dpkg`

    ```sh
    dpkg --get-selections | less
    ```

- List all the folders, in ascending order of size

    ```sh
    du -h --max-depth=1 | sort -h
    ```
