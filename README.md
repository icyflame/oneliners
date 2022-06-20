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

- Replace target regular expression with replacement (using captured groups) with GNU `sed` (or
  `gsed`)

  ```sh
  gsed -ie "s/some \(text\)/\U\1/g" *.xml
  ```

  `sed` uses basic regular expressions by default, hence the group capturing parentheses
  [MUST](http://stackoverflow.com/a/24717687/2080089) be escaped. `sed`'s options to use extended
  regexp can be turned on using the `-E` flag. The above command would become:

  ```sh
  $ echo "some text here" | gsed -E "s/some (text)/\U\1/g"
  TEXT here
  ```

  `sed` can be used with characters other than `/` as the separating character for the
  commands. This is clearer in some contexts. Especially, when the text to be replaced has a forward
  slash itself.

  ```sh
  $ echo "https://example.com" | gsed 's#/#-#g'
  https:--example.com
  ```

  `sed` prints the pattern space (i.e. input stream) by default. This can be annoying if you want to
  print the output of a transform _only_ when the pattern that we are searching for exists in the
  input.

  ```sh
  $ echo "this is not a URL" | gsed 's#http://#https://#g'
  this is not a URL

  $ echo "http://example.com" | gsed 's#http://#https://#g'
  https://example.com
  ```

  Compare this confusing output which transforms the input if the input is found and prints it
  without applying the transform when the pattern is not found, to the following output which prints
  something to the console **only** when the transform is applied. **Note** that here the expression
  _must_ have the `p` command, which tells `sed` to explicitly print the output of the previous
  `s///g` command.

  ```sh
  $ echo "this is not a URL" | gsed -n 's#http://#https://#gp'
  # Empty output

  $ echo "http://example.com" | gsed -n 's#http://#https://#gp'
  https://example.com
  ```

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

- Encrypt a file using the ChaCha20 stream cipher (using `openssl`)
  ```sh
  KEY=$(uuidgen); echo $KEY > /tmp/1.key
  echo "test" > /tmp/1.in
  openssl enc -chacha20 -kfile /tmp/1.key -pbkdf2 -base64 -out /tmp/1.out < /tmp/1.in;
  ```

- Using `pdftk` to concatenate multiple files ("Stapler, hole-punch, binder for PDF files")
  ```sh
  # Append multiple PDF files together
  pdftk 1.pdf 2.pdf cat output output.pdf
  ```

- Using `pdftk` to rotate pages in PDF files
  ```sh
  # Rotate a range of pages in PDF files. Other files will be left unchanged and passed through
  # as-is.
  #
  # This will create an output PDF with the second and third pages rotated 90 degrees
  # anti-clockwise. All other pages in in.pdf will retain their original orientation.
  $ pdftk in.pdf rotate 2-3left output out.pdf

  # Rotate pages and concatenate multiple PDFs simultaneously
  #
  # This will create output PDF with 2 pages: the first page of the file in1.pdf, rotated left (90 degrees
  # CCW) and the first page of in2.pdf, rotated right (90 degrees CW)
  $ pdftk A=in1.pdf B=in2.pdf cat A1left B1right output out.pdf
  ```


- Using `pdftk` to markup PDF files
  ```sh
  # Markup PDF files
  ## Burst open PDF file into its composite pages
  pdftk input.pdf burst

  ## Markup each PDF file using Gimp
  gimp pg*.pdf

  ## Put the PDF files back together using pdftk
  pdftk pg*.pdf cat output output.pdf
  ```

- Using `qpdf` to decrypt files which have [a password](https://askubuntu.com/a/828727)
  ```sh
  qpdf -password=<your-password> -decrypt /path/to/secured.pdf out.pdf
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

- Get the TLS (HTTPS) certificate for a domain
  ```sh
  # Print the certificate as human readable text
  openssl s_client -tls1_2 -connect duckduckgo.com:443 < /dev/null 2> /dev/null | openssl x509 -text

  # Print the certificate for a given host name if the domain has multiple TLS
  # hosts
  openssl s_client -tls1_2 -connect duckduckgo.com:443 -servername duckduckgo.com < /dev/null 2> /dev/null | openssl x509 -text

  # Check if the certificate is going to expire in the next 30 seconds
  openssl s_client -tls1_2 -connect duckduckgo.com:443 < /dev/null 2> /dev/null | openssl x509 -checkend 30
  ```

- Find a regular expression and print the first captured group based on some condition (using Perl)

  ```sh
  $ cat <<EOF | perl -lane 'm!count: ([0-9]+)! and $1 > 10 and print $1'
  count: 50
  count: 9
  test: 10
  EOF
  50
  ```

  We are using 4 Perl options.

  1. `-l`: Enable line-by-line processing
  2. `-a`: Enable autosplit mode (the input line is split at whitespace characters and the result is
     stored in the array `@F`
  3. `-n`: The given expression is put inside a loop in which each line is processed
     sequentially. Using this option, the final program is equivalent to: `while(<>) { ... given
     expression ... }`
  4. `-e`: Provide a single line of the Perl script

[1]: https://www.ivarch.com/programs/quickref/pv.shtml
