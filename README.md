# coopcreds.com

coopcreds.com is a Hugo site deployed on the same ubuntu server as community.coopcreds.com, a standard Discourse Docker installation. There's an external nginx (external to the Discourse docker image) running that handles requests for both coopcreds.com and community.coopcreds.com. There's also a dockerized postfix server handling inbound email (see below).

## Updating coopcreds.com

### Setup

> You ssh access to coopcreds.com to deploy changes.

There's bare clone of this repository on the production server, with a ``post-receive`` hook that will handle the compliation of hugo's static assets. You just need to add the production repository as a remote and push to it. The hook will take care of the rest.

as a remote

```
git remote add production USERNAME@coopcreds:coopcreds.com.git
git ls-remote ## to verify it's added
```

You should see an output that looks something this
```
103902f5d448cf57425bd6830e544128d9522c51    HEAD
...
```

### Deploy

```
git push production main
```

## Handling email

There's a postfix server running in the container ``mail-receiver``. It handles both incoming mail for Discourse (i.e. ``community.coopcreds.com``) and email forwarding for some ``alias@coopcreds.com`` addresses. 


### Add an alias

New ``alias@coopcreds.com`` addresses can be added as follows.

First, add the mapping to the [virtual alias map](http://www.postfix.org/postconf.5.html#virtual_alias_maps)

```
vi /var/discourse/shared/mail-receiver/etc/virtual
```

Then re-map the virtual alias map

```
/var/discourse/launcher enter mail-receiver
postmap /etc/postfix/shared/virtual
exit
```

Finally, restart the mail-receiver
```
/var/discourse/launcher restart mail-receiver
```

### Adding a group alias

Group aliases should be handled via Discourse groups, meaning that the alias map needs to point to an existing group email alias on Discourse (i.e. that's already been setup in Discourse). The group email in Discourse should be set using the apex domain version. An example is given below.

#### Alias map (in the postfix virtual alias map)

```
contact@coopcreds.com contact@community.coopcreds.com
```

#### Group email (in Discourse group settings)

```
contact@coopcreds.com
```
