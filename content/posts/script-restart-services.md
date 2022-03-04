---
title: "Script restart services"
date: 2017-08-18T22:29:53+02:00
draft: false
tags: ['linux', 'bash', 'script']
---

Sometimes, we find ourselves having to pilote infrastructures services supported by panels managements and ... between us, I have a holy horror of Plesk, Cpanel, Ispconfig, Webmin ...
Especially when under certain circumstances some components are falling down.
It's often badly done and messy that it's difficult to find your way around. Between abobinable logs management (yes, I already found php logs in syslog), a non-existent server optimization... well I stop here because it is not the purpose of this post.

In fact, what happened lately is that I discovered on a "certain version" of a "certain panel" that I won't write here, php-fpm is crashing miserably and randomly. Of course, just to make it funny, this incident leads to a service outage for about 200 users, cool no?

So, at some point you have to say f*ck and drive the mess properly, so you get the bash out.
So yes, you tell me, why bother, just use a service like monit? Well I tested but no.

So, just to make it right, we start our script the following way.

```
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

#/ Usage: ./main.sh
#/ Description: This script monitore services
#/ Examples: ./main.sh
#/ Options:
#/   --help: Display this help message
usage() { grep '^#/' "$0" | cut -c4- ; exit 0 ; }
expr "$*" : ".*--help" > /dev/null && usage

readonly LOG_FILE="/tmp/$(basename "$0").log"
info()    { echo "$(date -u) [INFO]    $*" | tee -a "$LOG_FILE" >&2 ; }
warning() { echo "$(date -u) [WARNING] $*" | tee -a "$LOG_FILE" >&2 ; }
error()   { echo "$(date -u) [ERROR]   $*" | tee -a "$LOG_FILE" >&2 ; }
fatal()   { echo "$(date -u) [FATAL]   $*" | tee -a "$LOG_FILE" >&2 ; exit 1 ; }

# Debug
# Cleaning if exit
cleanup() {
    rm "$LOG_FILE"
}
trap cleanup EXIT
```

So far, nothing extraordinary, we have our shebang, different logging functions and a clean service with trap.

As I like to be alerted of the actions of my scripts, we're going to declare a mail sending function:
```
# Func mail
sendMail() {
    echo "$(date) : $1 restarted on $HOSTNAME" | mail -s "$1 restarted on $HOSTNAME" "my@mail"
}
```

We will then declare our function for a given service (at random, a service that never falls...):
```
# Func php-fpm
php-fpm() {
if pgrep php5-fpm > /dev/null
then
    info "Php-FPM service is up."
:
else
       warning "Php-FPM service down, restarting it !"
       if /etc/init.d/php5-fpm restart > /dev/null
       then
           info "Php-FPM service restarted"
           sendMail "php-FPM"
       fi
fi
}
```

Here we check if the service has an active pid, in which case it is restarted, logged and sent a mail.

Repeat the operation with the other services involved if necessary and call the main:

```
if [[ "${BASH_SOURCE[0]}" = "$0" ]]; then
    info "Starting..."

    while true; do

        # Functions call
        php-fpm
        sleep 5
        # vos autres fonctions ici

    done

fi
```

Full version :

```
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

#/ Usage: ./main.sh
#/ Description: This script monitore Plesk Web services
#/ Examples: ./main.sh
#/ Options:
#/   --help: Display this help message
usage() { grep '^#/' "$0" | cut -c4- ; exit 0 ; }
expr "$*" : ".*--help" > /dev/null && usage

readonly LOG_FILE="/tmp/$(basename "$0").log"
info()    { echo "$(date -u) [INFO]    $*" | tee -a "$LOG_FILE" >&2 ; }
warning() { echo "$(date -u) [WARNING] $*" | tee -a "$LOG_FILE" >&2 ; }
error()   { echo "$(date -u) [ERROR]   $*" | tee -a "$LOG_FILE" >&2 ; }
fatal()   { echo "$(date -u) [FATAL]   $*" | tee -a "$LOG_FILE" >&2 ; exit 1 ; }

# Debug
# Cleaning if exit
# cleanup() {
#     rm "$LOG_FILE"
# }
# trap cleanup EXIT


# Functions 

    # Func mail
    sendMail() {
        echo "$(date) : $1 restarted on $HOSTNAME" | mail -s "$1 restarted on $HOSTNAME" "my@mail"
    }

    # Func nginx
    nginx() {
    if pgrep nginx > /dev/null
    then
#        info "Nginx service is up."
    :
    else
            warning "Nginx service down, restarting it !"
            if /etc/init.d/nginx restart > /dev/null
            then
                info "Nginx service restarted"
                sendMail "nginx"
            fi
    fi
    }

    # Func php-fpm
    php-fpm() {
    if pgrep php5-fpm > /dev/null
    then
#        info "Php-FPM service is up."
    :
    else
           warning "Php-FPM service down, restarting it !"
           if /etc/init.d/php5-fpm restart > /dev/null
           then
               info "Php-FPM service restarted"
               sendMail "php-FPM"
           fi
    fi
    }

    # Func mysql
    mysql(){
    if pgrep mysql > /dev/null
    then
#        info "Mysql service is up."
    :
    else
        warning "Mysql service down, restarting it !"
        if /etc/init.d/mysql restart > /dev/null 
        then
            info "Mysql service restarted"
            sendMail "mysql"
        fi
    fi
    }

    # Func apache
    apache(){
    if pgrep apache2 > /dev/null
    then
#        info "Apache2 service is up."
    :
    else
        warning "Apache2 service down, restarting it !"
        if /etc/init.d/apache2 restart > /dev/null 
        then
            info "Apache2 service restarted"
            sendMail "apache2"
        fi
    fi
    }

if [[ "${BASH_SOURCE[0]}" = "$0" ]]; then

    # If root :
	if [[ $EUID -ne 0 ]]; then
    	echo "This script must be run as root"
    	exit 1
    fi

    info "Starting..."

    while true; do

        # Functions call
        nginx    	
        sleep 5
        php-fpm
        sleep 5
        mysql
        sleep 5
        apache
        sleep 5

    done

fi
```

Well, it's nothing too crazy, but I find that doing things manually really helps you target exactly what you want to do.

As usual, the sources are available!

In a future post, we'll see how to factorize all this script to avoid repetition and make it more readable.

See you soon!
