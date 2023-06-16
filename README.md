# test-scalingo-web

Simple nginx app using Scalingo nginx-buildpack

This nginx config enable the following feature:
 - modsecurity enable and one rule (for testing purpose)
 - restrict access to /protected url based on:
   - IP/CIDR allow deny list (`ALLOWED_IPS_LIST_BASE64`)
   - user/password list (basic auth) (`ALLOWED_IPS_LIST_BASE64`)

## modsecurity config
modify `modsecurity_rules.conf` or `modsecurity_rules.d`

```
# enable environment variable in env app
scalingo --app test-nginx env-set \
   ENABLE_MODSECURITY=true \
   MODSECURITY_AUDIT_LOG_LEVEL=Off \
   MODSECURITY_DEBUG_LOG_LEVEL=0 \
```

## Protected Location based on IP/cidr
In this example, /protected is restricted by IP/CIDR list

Restricted IP/CIDR List is defined in `ALLOWED_IPS_LIST_BASE64` environment in the scalingo app

`ALLOWED_IPS_LIST_BASE64` contains base64 encoded file of ip/cidr allow/deny list.

To create ALLOWED_IPS_LIST_BASE64,
- Create allow-ip file , one line per allow|deny and IP or CIDR; (don t forget the ';' at the end of lines)
- Encode the file in base64
- Save the encoded result in scalingo env variable `ALLOWED_IPS_LIST_BASE64`
- If variable is empty: default is `deny all;`

```
# syntax : [IP or CIDR] [allow|deny];
#
# Allow all
#
cat <<EOF > allow-ip.conf
allow 0.0.0.0/0;
EOF
ALLOWED_IPS_LIST_BASE64="$(cat allow-ip.conf |base64)"
```

```
# Allow specific CIDR or IP and
# encode in base64
cat <<EOF |base64
allow 172.16.1.1;
allow 10.0.0.0/8;
allow 192.168.10.0/24;
EOF
```
Save encoded value in the environment part of the app
```
#
scalingo --app test-nginx env-set ALLOWED_IPS_LIST_BASE64="$ALLOWED_IPS_LIST_BASE64"
```

Quick one liner
```
scalingo --app test-nginx env-set ALLOWED_IPS_LIST_BASE64="$(cat <<EOF |base64
allow 0.0.0.0/0;
EOF
)"
```

## Protected Location based on basic auth (user/passwd) list

In this example, /protected is restricted by htpasswd list

Restricted htpasswd List is defined in `ALLOWED_USERS_LIST_BASE64` environment

`ALLOWED_USERS_LIST_BASE64` contains base64 encoded file of users:cryptpasswd


To create `ALLOWED_USERS_LIST_BASE64`,
- Create allow-users file , one line per user:cryptedpasswd
- Encode the file in base64
- Save the encoded result in scalingo env variable `ALLOWED_USERS_LIST_BASE64`
- If variable is empty: no auth

```
# generate htpasswd file, with htpasswd command
#
# or use the following command for one user
#    replace $USER and $USER_PASSWORD
#
echo "$USER:$(openssl passwd -1 -salt $(openssl rand -base64 6) $USER_PASSWORD)"
```

```
# encode the file in base64
ALLOWED_USERS_LIST_BASE64="$(cat allow-user.conf |base64)"
# set env app
scalingo --app test-nginx env-set ALLOWED_USERS_LIST_BASE64="$ALLOWED_USERS_LIST_BASE64"
```
