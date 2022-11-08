---
title: TeamCity Tips and Tricks
layout: post
post-image: "/assets/images/blog/teamcity.jpg"
description: Tips and tricks on how to use and configure TeamCity projects, build configurations and templates
tags:
- howto
- blog
- devsecops
- teamcity
- akamai
- hashicorp-vault
---
- [Overview](#overview)
- [Saving Multiline Parameter Value to File](#saving-multiline-parameter-value-to-file)
- [TeamCity Parameter Setup Using Service Messages](#teamcity-parameter-setup-using-service-messages)
- [Misbehaving Proxy Server](#misbehaving-proxy-server)

# Overview

[TeamCity](https://www.jetbrains.com/teamcity/) is the the Hassle-Free CI/CD Tool by JetBrains. It can be used to build CI/CD pipelines that integrate with other DevOps/DevSecOps tools like GitHub, HashiCorp Vault etc.
This blog post covers various useful tips and tricks that can maximize the TeamCity usage experience.

# Saving Multiline Parameter Value to File

Assuming that a parameter named `env.ROOT_CERT` is used to store a root certificate in the following format:

```
-----BEGIN CERTIFICATE-----
abcdefghijklmnopqrstuvwxyz
12345678901234567890123456
-----END CERTIFICATE-----
```

and if the following command in the Custom script of the Command Line Runner type is used to write the value to a file
```
echo $ROOT_CERT > root_cert.crt
```

then the integrity of the certificate value is not maintained, i.e the `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` are not shown in `root_cert.crt`.
<br>
The solution to this issue is to put `$ROOT_CERT` in the double quotes `""` as follows:

```
echo "$ROOT_CERT" > root_cert.crt
```

# TeamCity Parameter Setup Using Service Messages

TeamCity [Service Messages](https://www.jetbrains.com/help/teamcity/service-messages.html) allows for setting up parameters of a TeamCity build configuration (a.k.a CI/CD pipeline) from a build step that runs within a Docker container. This is needed if there is a requirement of retaining some results produced by that build step for later use in subsequent build steps as the whole runtime environment is destroyed together with the Docker container when the build step finishes.
<br>
Let's talk about a real-world scenario and how to achieve the goal. In that scenario, the TeamCity pipeline is required to perform the followings:

- Retrieving Akamai API client credentials from a HasiCorp Vault key path
- Writing those credentials to a TeamCity environment parameter/variable called `env.AKAMAI_EDGERC` to be accessed by the subsequent build steps
<br>
To retrieve the credentials from the Vault, the following Python snippet can be used in the Custom script of the Command Line Runner type:

```Python
cat > get-credentials.py <<EOF
secret = ldap_client.read_secret('%vault.akamai.edgerc.path%')
print("##teamcity[setParameter name='env.AKAMAI_EDGERC' value='{}']".format(secret))
EOF

python get-credentials.py
```

<br>
The Akamai API client credentials have multiple lines with some special characters from TeamCity's point of view like the following:
```
[default]
client_secret = abc123=
host = abc.akamai.net
access_token = abc123
client_token = abc123
```

<br>
If the credentials are stored as they are in the Vault, the above `print` statement will generate this error message:

```
teamcity[setParameter name='env.AKAMAI_EDGERC' value='[default]
Error while parsing TeamCity service message: Value should end with "'". Valid service message has a form of "##teamcity[messageName name1='escaped_value' name2='escaped_value']" where escaped_value uses substitutions: '->|', [->|[, ]->|], |->||, newline->|n
```

To resolve this error, the special characters need to be escaped following the [Escaped Values](https://www.jetbrains.com/help/teamcity/service-messages.html#Escaped+Values) rules. As the result, the credentials need to be stored in the Vault in the following format:

```
|[default|]|n|rclient_secret = abc123=|n|rhost = abc.akamai.net |n|raccess_token = abc123|n|rclient_token = abc123
```

# Misbehaving Proxy Server

The situation is summarized as follows:

- Terraform code is executed in a Docker container in one of the TeamCity build step
- The proxy password has some special characters and is stored in the environment parameter `env.PROXY_PASSWORD`
- In that build step, the proxy environment variables are set up correctly as follows:
  ```
  export http_proxy=http://"$PROXY_USERNAME":"PROXY_PASSWORD"@$PROXY_URL
  export https_proxy=http://"$PROXY_USERNAME":"PROXY_PASSWORD"@$PROXY_URL
  export no_proxy="*.local,*.internal"
  ```
- Terraform commands `terraform init` and `terraform plan` are in that build step
<br>
When the Terraform commands `terraform init` and `terraform plan` are executed, the build fails and generates the following error:
```
Error: fetching groups: request failed: Get "<Terraform API endpoint>": proxyconnect tcp: dial tcp: lookup http on <some IP address>: server misbehaving
```
<br>

The solution is mentioned in this article [Set/Export: http_proxy With Special Characters In Password Under Unix / Linux](https://www.cyberciti.biz/faq/unix-linux-export-variable-http_proxy-with-special-characters/). What need to be done are:

- identify special characters and convert them to hexadecimal like `@ => 0x40` using some online tools like [this one](https://onlineunicodetools.com/convert-unicode-to-hex)
- then replace `0x` with `%`. For example, `@` will be come `%40`
