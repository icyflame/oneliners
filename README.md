# one-liners

> Bash one-liners that are always useful :heart: :heart:

### TOC

- [Why](#why)
- [List](#init)
  - [Concepts](#concepts)
  - [Commands](#commands)
  - [Docker](#docker)

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

- Removes a kernel module and then re-adds it. Doing this for `usbhid`, fixes
problems in Ubuntu 16.04 LTS related to the mouse or other USB peripherals
  ```sh
  sudo modprobe -r usbhid && sleep 1 && sudo modprobe usbhid
  ```

- Convert between AV formats using `ffmpeg`
  ```sh
  ffmpeg -i input_file.mp4 output_file.mp3
  ```

  Common formats are accepted, such as: avi, wav, mp3, mp4, mkv etc. (Audio,
  Video streams etc are inferred on the basis of the output file name)

- Watch a file get appended using `tail`
  ```sh
  tail -f ~/test.log
  ```

- Use `vlc` to play a random file from a recursive list of _all_ files
  ```sh
  vlc "`find . -not -type d | shuf | head -n1`"
  ```

  **Note:** It is inherently assumed that the subtree of the folder from which
  this command is being run doesn't contain any files that are not playable by
  VLC, such as txt, doc, etc. This oneliner _can_ be enhanced to ensure that
  only video files are found and played. If you find an elegant way to do
  that, please open a PR!

- List all the files in a folder except the most recently modified one
  ```sh
  ls -tr | head -n $((`ls -tr | wc -l`-1))
  ```

- Move current window to the top in `tmux`
  ```sh
  move-window -t 0
  ```

- Swap windows N1 and N2 inside `tmux`
  ```sh
  swap-window -t 0
  swap-window -s N1 -t N2
  ```

- Show progress while creating a gzipped archive using tar and gzip - using [pv][1]
  ```sh
  tar cf - FOLDER | pv -cN compression -s `du -sb FOLDER | cut -f1` | gzip -9 > 1A.tar.gz
  ```

- Show progress while encrypting a file using GPG
  ```sh
  pv FILE | gpg --symmetric --passphrase "test" > FILE.gpg
  ```

- Re-attach the top session of `screen` in the output of `screen -ls`
  ```sh
  screen -r `screen -ls | head -n-1 | tail -n-1 | awk '{ print $1 }'`
  ```

#### Docker

- Serving the current directory on the local network using `nginx`
  ```sh
  $ cat test.conf
  server {
      listen       9091;
      server_name  localhost;

      location / {
          root   /var/www/html;
          index  index.html index.htm;

          # ngx_http_access_module
          allow 10.0.0.0/24;
          deny all;
      }
  }

  $ docker run -p 9090:9091 \
    -v "`pwd`:/var/www/html" \
    -v "`pwd`/test.conf:/etc/nginx/conf.d/localserver.conf" \
    -d --name localserver nginx:latest
  ```

- Converting a markdown file to an HTML file
  ```sh
  docker run \
    -v `pwd`:/source jagregory/pandoc \
    -f markdown -t html5 \
    scratch/scratch-2019-09-17-22-33-55-z-review.md -o z-review.html
  ```

- Run `htop` on a host
  ```sh
  docker run -it --pid=host jonbaldie/htop
  ```

- Run a squid proxy server on port 3128
  ```sh
  # Prepare the configuration file
  docker run --rm sameersbn/squid \
    cat /etc/squid/squid.conf > squid.conf
  # Edit the configuration file
  # Start the proxy server using this configuration
  docker run --name squid -d --restart=always \
    --publish 3128:3128 \
    -v "$PWD/squid.conf":/etc/squid/squid.conf \
    sameersbn/squid
  ```

- Get the TLS certificate for a domain
  ```sh
  # Print the certificate as human readable text
  openssl s_client -tls1_2 -connect duckduckgo.com:443 < /dev/null 2> /dev/null | openssl x509 -text

  # Print the certificate for a given host name if the domain has multiple TLS
  # hosts
  openssl s_client -tls1_2 -connect duckduckgo.com:443 -servername duckduckgo.com < /dev/null 2> /dev/null | openssl x509 -text

  # Check if the certificate is going to expire in the next 30 seconds
  openssl s_client -tls1_2 -connect duckduckgo.com:443 < /dev/null 2> /dev/null | openssl x509 -checkend 30
  ```

[1]: https://www.ivarch.com/programs/quickref/pv.shtml
