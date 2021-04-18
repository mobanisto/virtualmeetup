# WorkAdventure

## Test locally

Go to `workadventure-xce/vagrant` and follow instructions from the local
README.md file.
You will use this configuration file: `contrib/docker/env.devel`.

## Run in production

Go to `workadventure-xce` and run `./run-server.sh`.
It will use this configuration file: `contrib/docker/env.server`.

## TURN server (coturn)

We had some trouble setting up TURN properly which is required to make
the video-communication in the small groups possible for people behind
firewalls. A technology called WebRTC is used for making that communication
possible and a TURN server can be used to establish connections for users
who cannot connect directly due to firewall restrictions.

### Intro

In general you can install coturn on your host OS, or as yet another
service in your docker composition. This section currently assumes your
setting it up on your host OS.

### Make certificates readable for turnserver user

You'll need TLS certificates to make coturn use a safe transport. You
can configure LetsEncrypt certifactes using nginx and certbot.
(TODO: add notes on how to do that).
Make sure the certificates are copied to a location readable by coturn
such as `/etc/coturn/certs` using a renewal hook in
`/etc/letsencrypt/renewal-hooks/deploy/coturn-certbot-deploy.sh` so that
each time the certificates get updated, they are updated in that location,
too. (Normally, certificates are only readable by `root` and coturn is run
by system user `turnserver` who cannot access those certificates)

[Read more in this StackOverflow answer](
https://serverfault.com/a/984575/376285)

### Test using Jitsi

We found it useful to test coturn using Jitsi and then moving on to
WorkAdventure knowing that the setup was generally working.
What's also helpful is, when installing Jitsi, coturn is automatically
installed and configured in such a way that it can be used with
WorkAdventure.

What you still need to do is copy the authentication secret that is used
for generating valid temporary username/password combinations from the
coturn configuration to the WorkAdventure configuration.

To do so, open `/etc/turnserver.conf` and copy the value of property
`static-auth-secret` to `contrib/docker/env.devel` and/or
`contrib/docker/env.server`.

For example if `/etc/turnserver.conf` contains this line:

    static-auth-secret=YourCoturnStaticAuthSecret

then put this line into `contrib/docker/env.devel`:

    TURN_STATIC_AUTH_SECRET=YourCoturnStaticAuthSecret

### Force TURN server usage

To simulate a user that cannot establish a direct connection, it makes sense
to make that reproducible. It is possible to make Firefox only go through
relays instead of connecting to peers directly. Within Firefox, navigate
to `about:config` and toggle the `media.peerconnection.ice.relay_only` to
true.

[Read more in this StackOverflow answer](
https://stackoverflow.com/a/39490790/1268759)

### Reading the turn server log

The log output from coturn can be seen like this:

    journalctl -u coturn -b -f

also the server can be run directly using the `turnserver` command instead
of using `service coturn start` and the output will be written to stdout.

### Testing with a minimal WebRTC example

I also found it useful to test using a minimal WebRTC example like this
one: https://github.com/shanet/WebRTC-Example
